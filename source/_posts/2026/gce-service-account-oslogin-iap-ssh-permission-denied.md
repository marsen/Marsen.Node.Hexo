---
title: "[實作筆記] Service Account 用 IAP tunnel SSH 進 GCE，明明權限都給了還是 Permission denied"
date: 2026/07/20 18:25:10
tags:
  - 實作筆記
---

## 前情提要

在幫 AIris（LINE 分身橋接專案）設定 CD pipeline，讓 GitHub Actions 可以用 Workload Identity Federation（WIF）換一個 service account 的身分，透過 IAP tunnel SSH 進 GCE VM 部署。權限明明照著給了，IAM 也三種方式驗證過都說「有權限」，VM 卻死咬著 `Permission denied`。查到最後發現是漏了一個沒那麼直覺的角色綁定，而且我自己中途還推了一個錯誤方向、代價還不小（要停機）。記錄一下踩坑過程。

## 建置過程：WIF + service account + IAP

建了一個 WIF pool/provider，綁定到 GitHub repo，再建一個專用的 service account `airis-deploy`，綁了兩個角色：

- `roles/iap.tunnelResourceAccessor`（開 IAP tunnel）
- `roles/compute.osAdminLogin`（OS Login，含 sudo）

VM 本身也確認過開著 OS Login（`enable-oslogin=TRUE`）。照著這樣設定，理論上 GitHub Actions 認證後應該能直接：

```bash
gcloud compute ssh ai-butler-vm00 --zone=us-east1-b --tunnel-through-iap \
  --impersonate-service-account="airis-deploy@PROJECT.iam.gserviceaccount.com" \
  --command="whoami"
```

## 症狀：IAM 說有權限，VM 說沒有

實際跑起來，SSH 直接被拒絕：

```text
sa_103328536429727217784@compute.xxx: Permission denied (publickey).
```

VM 上的 sshd log 更明確：

```text
sshd: google_authorized_keys: OS Login user sa_103328536429727217784 does not have login permission.
sshd: google_authorized_keys: Could not grant access to organization user: sa_103328536429727217784.
```

但問題是，我用三種不同方式直接跟 GCP API 確認過，這個 service account **明明有**登入權限：

```bash
# Cloud Resource Manager API
curl -X POST "https://cloudresourcemanager.googleapis.com/v1/projects/PROJECT:testIamPermissions" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"permissions":["compute.instances.osLogin","compute.instances.osAdminLogin"]}'
# → 兩個權限都回來了

# Compute API，instance 層級
curl -X POST ".../instances/ai-butler-vm00/testIamPermissions" ...
# → 一樣都有

# OS Login API 的 getLoginProfile
gcloud compute os-login describe-profile --impersonate-service-account="airis-deploy@..."
# → posixAccounts、sshPublicKeys 都在，account 是 primary
```

三個角度都確認「這個身分有權限、SSH key 也確實註冊好了」，但 VM 端就是不認。中途試過：多等幾分鐘讓 IAM propagate、把 binding 移除再重新加一次想強制刷新，都沒用。

## 錯誤推論：以為是 VM 的 OAuth scope 不夠

看了一下 VM 自己掛的預設 service account（不是新建的 `airis-deploy`，是 VM 出生就有的那個）的 OAuth scope：

```text
devstorage.read_only
logging.write
monitoring.write
pubsub
service.management.readonly
servicecontrol
trace.append
```

沒有任何 compute 相關的範圍。我當時的推論是：VM 收到登入請求時，得反過來拿自己的身分去問 Google「這個新身分有沒有權限」，但自己的 scope 不夠，這個反查就默默失敗了。

這個推論**沒有查證過**，而且修法的代價不小——GCE 的 instance scope 只能在**停機狀態**下改，代表要把這台跑著 n8n 的 VM 停機、改設定、重開機。

Marsen 問了一句「這是實驗還是確定的修改？」，這句話讓我停下來——在建議使用者為了一個沒把握的假設去承擔停機成本之前，應該先查證，不是先動手。

## 真正的根因：漏了一個角色，而且要綁在別的資源上

用 WebSearch 查 GCP 官方文件才找到關鍵：

> If a user is granted the `roles/compute.osLogin` access role and the authorization output returns `{"success": false}`, this indicates that the user might be missing the `roles/iam.serviceAccountUser` permission for the service account associated with the compute instance.

重點是**「for the service account associated with the compute instance」**——這個角色要綁在「VM 本身掛載的那個 service account」上，member 是我們的 `airis-deploy`，不是綁在 project 或 VM instance 這兩個我原本驗證過的資源上：

```bash
gcloud iam service-accounts add-iam-policy-binding \
  PROJECT_NUMBER-compute@developer.gserviceaccount.com \
  --member="serviceAccount:airis-deploy@PROJECT.iam.gserviceaccount.com" \
  --role="roles/iam.serviceAccountUser"
```

補上這個綁定，完全不用停機，等了 1-2 分鐘傳播後，SSH 跟 sudo 都成功了。

## 為什麼三個 API 驗證都測不出這個問題

因為那三個 API 檢查的都是「這個身分有沒有被授權」，資源分別是 project、VM instance、OS Login profile——都指向同一件事：`airis-deploy` 這個身分本身夠不夠格登入。

但漏掉的那個角色，檢查的是完全不同的資源：**VM 的 service account**，問的是另一個問題——「`airis-deploy` 能不能『使用』VM 這個身分登入的這整條鏈路」。這是 OS Login 底層驗證流程裡的一個環節，不在我原本驗證的三個檢查範圍內，所以怎麼測都測不出來。

## 小結

- IAM 權限「確認生效」跟「這個資源實際能不能用」是兩件事，尤其是 OS Login 這種還牽涉到 VM 自己反查權限的機制
- 缺的角色綁在完全不同的資源上（VM 的 service account），不是我以為的 project 或 VM instance，難怪測不出來
- 遇到「權限都給了還是不通」，先查官方 troubleshooting 文件，不要憑經驗推論一個代價更高（尤其是要別人承擔停機成本）的方案
- `roles/iam.serviceAccountUser` 這個角色常常是這類「明明權限給了卻卡住」問題的漏網之魚

## 資料來源

- [Set up OS Login | Compute Engine](https://docs.cloud.google.com/compute/docs/oslogin/set-up-oslogin)
- [Troubleshooting OS Login | Compute Engine](https://cloud.google.com/compute/docs/troubleshooting/troubleshoot-os-login)

(fin)
