---
title: "個人自動化平台(二) Cloudflare Access & Cloudflare Tunnel"
date: 2026/04/17 14:54:04
tags:
  - 實作筆記
---

## 前情提要

上一篇在 GCP 免費 VM 上把 n8n 跑起來了，但還沒辦法從外部存取。

這篇用 cloudflared tunnel 解決這個問題。

好處是：

- 不用改 GCP 防火牆，不暴露任何 port
- 自動 HTTPS
- 可以加 Cloudflare Access（Google 帳號驗證）擋在前面

---

## 架構概觀

這篇要做的事情分兩層：

### 第一層：cloudflared tunnel

在 VM 裡跑一個 cloudflared process，它會主動連到 Cloudflare 的網路，建立一條加密的隧道。
外部流量進來的路徑是：

```text
使用者瀏覽器 → Cloudflare → cloudflared tunnel（VM 內）→ n8n（127.0.0.1:5678）
```

VM 不需要開任何防火牆 port，因為連線是由 VM 內部主動發起的。

### 第二層：Cloudflare Access（Zero Trust）

cloudflared tunnel 只負責轉流量，本身不做身份驗證，任何人都能連進來。
Cloudflare Access 是加在 tunnel 前面的門禁：

```text
使用者瀏覽器 → Cloudflare Access（驗證身份）→ cloudflared tunnel → n8n
```

只有通過驗證（這裡用 email OTP）的人才能到達 n8n。
這就是「Zero Trust」的概念：不信任任何人，每次都要驗證身份。

## 這篇的步驟順序

1. 安裝 cloudflared
2. 登入 Cloudflare、建立 tunnel、設定 config、新增 DNS
3. 設定 Cloudflare Access（**先做，tunnel 上線前就有門禁**）
4. 設定 systemd 並啟動 tunnel

---

## 步驟零：安裝 cloudflared

在 VM 上執行：

```bash
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null
echo 'deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared bookworm main' | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt update && sudo apt install cloudflared
```

看到 `Setting up cloudflared` 就安裝成功了。

---

## 命名決策

這台 VM 不只跑 n8n，之後可能還有其他自動化服務。
所以 tunnel 和 subdomain 都以機器用途命名，而不是以某個服務命名。

以這篇為例，選 `butler`（管家），因為這台機器的定位是「幫你自動處理事情的後台」。

- `<TUNNEL_NAME>` = `butler`
- `<YOUR_DOMAIN>` = `marsen.me`
- 對外網址：`butler.marsen.me`

之後如果加新服務，在 config 裡多加一條 ingress 規則就好。

---

## 步驟一：登入 Cloudflare

在 VM 上跑：

```bash
cloudflared tunnel login
```

它會印出一個 URL，複製到瀏覽器，選你要授權的 domain（這裡是 `<YOUR_DOMAIN>`）。

授權完成後 cert 會自動存到：

```text
~/.cloudflared/cert.pem
```

---

## 步驟二：建立 Tunnel

```bash
cloudflared tunnel create <TUNNEL_NAME>
```

成功後會顯示 tunnel ID（UUID 格式），同時在 `~/.cloudflared/` 產生一個 credentials JSON 檔。

```text
Tunnel credentials written to /home/USER/.cloudflared/<tunnel-id>.json
Created tunnel <TUNNEL_NAME> with id <tunnel-id>
```

把 tunnel ID 記下來，後面要用。

---

## 步驟三：建立 config 檔

```bash
cat > ~/.cloudflared/config.yml << 'EOF'
tunnel: <your-tunnel-id>
credentials-file: /home/<USER>/.cloudflared/<your-tunnel-id>.json

ingress:
  - hostname: <TUNNEL_NAME>.<YOUR_DOMAIN>
    service: http://127.0.0.1:5678
  - service: http_status:404
EOF
```

注意最後的 `EOF` 必須從行首開始，前面不能有任何空格。

> **為什麼用 `127.0.0.1` 而不是 `localhost`？**
> `localhost` 在某些環境會被解析成 IPv6 的 `[::1]`，但 n8n 預設只監聽 IPv4。
> 直接寫 `127.0.0.1` 避免這個問題。

其中 `5678` 是 n8n 在 Docker 容器裡對外暴露的 port。

可以用以下指令驗證 config 是否正確：

```bash
cloudflared tunnel ingress validate
```

看到 `OK` 就是正確。

---

## 步驟四：新增 DNS 記錄

```bash
cloudflared tunnel route dns <TUNNEL_NAME> <TUNNEL_NAME>.<YOUR_DOMAIN>
```

這會在 Cloudflare DNS 自動加一筆 CNAME，指向你的 tunnel。

---

## 步驟五：設定 Cloudflare Access

**在啟動 tunnel 之前先設好 Access**，這樣 n8n 上線的那一刻就已經有門禁，不會有公開暴露的空窗期。

> **注意：不要用 Onboarding 精靈。** Zero Trust 首次進入會有引導精靈，精靈的「指派 Tunnel」步驟需要 tunnel 已在執行中才能選取。我們要先設 Access 再啟動 tunnel，所以改用手動建立 Application。

進 Cloudflare 主控台 → **Zero Trust** → **Access → Applications** → **+ Add an application** → 選 **Self-hosted**。

第一次進入 Zero Trust 需要選擇團隊名稱（之後可以改），Free 方案 50 人以下免費。

填入應用程式資訊：

| 欄位 | 填什麼 |
| --- | --- |
| Application name | `<APP_NAME>` |
| Subdomain | `<TUNNEL_NAME>` |
| Domain | `<YOUR_DOMAIN>` |

下一步設定 Policy：

| 欄位 | 填什麼 |
| --- | --- |
| Policy name | `allow` |
| Action | Allow |
| Selector | Emails |
| 值 | `<YOUR_EMAIL>` |

存檔。Access 現在保護 `<TUNNEL_NAME>.<YOUR_DOMAIN>`，只有通過驗證的 email 才能進去。

```text
使用者瀏覽器
    ↓ HTTPS
Cloudflare Access（驗證身份）
    ↓ 通過後
cloudflared tunnel（在 VM 裡跑）
    ↓ 本機連線
127.0.0.1:5678（n8n）
```

### 為什麼要在 tunnel 啟動前先做？

你一開 n8n 就會看到「Set up owner account」頁面，這個頁面是公開的。
如果沒有 Access 保護，任何人都能搶先填、搶走 owner 帳號。

---

## 步驟六：設定開機自動啟動並啟動 Tunnel

讓 cloudflared 在 VM 重開機後自動啟動，不用每次手動跑。

`cloudflared service install` 需要知道 config 在哪，但用 `sudo` 跑時 home 會變成 root 的，
所以要明確指定路徑。安裝後 systemd 會把 config 複製到 `/etc/cloudflared/config.yml`：

```bash
sudo cloudflared --config /home/<USER>/.cloudflared/config.yml service install
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```

把 `<USER>` 換成你的 OS Login 用戶名稱（Gmail 帳號把 `.` 和 `@` 換成 `_`，例如 `yourname_gmail_com`）。

看到 `active (running)` 就完成了。

這時開 `https://<TUNNEL_NAME>.<YOUR_DOMAIN>`，應該會先看到 Cloudflare Access 的驗證頁面。

之後 VM 重開機，cloudflared 和 n8n 都會自動恢復。

---

## 踩坑紀錄

### 踩坑一：瀏覽器顯示連線錯誤

**症狀**：tunnel 連上了（看到 `Registered tunnel connection`），但開瀏覽器只看到錯誤頁面。

瀏覽器的連線錯誤原因很多，不要猜。先查 n8n log：

```bash
sudo docker logs n8n --tail 20
```

根據 log 的錯誤訊息再對症下藥。

---

### 踩坑二：n8n 啟動失敗（permission denied）

**出現時機**：`docker logs` 看到 `EACCES: permission denied`。

**原因**：`docker run` 之前沒有預先建立 `~/.n8n`，Docker 自動用 root 身份建立這個目錄。
n8n 容器裡的 `node` 用戶（UID 1000）不是 root，沒有寫入權限，n8n 啟動就 crash。

**解法**：把目錄 owner 改成 UID 1000，再重建容器：

```bash
sudo chown -R 1000:1000 ~/.n8n
sudo docker rm -f n8n
sudo docker run -d \
  --name n8n \
  --restart unless-stopped \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  -e N8N_SECURE_COOKIE=false \
  docker.n8n.io/n8nio/n8n
```

**預防**：照第一篇步驟五，`docker run` 前先執行 `mkdir -p ~/.n8n && sudo chown 1000:1000 ~/.n8n`，就不會踩到這個坑。

n8n 映像檔本來就用非 root 的 `node` 用戶執行，不需要加 `--user`。

---

## 參考

- [個人自動化平台(一) n8n & GCP VM](/2026/n8n-1-in-gcp-free-tier-vm/)
- [個人自動化平台(三) 收集、處理、輸出：三層可插拔管道設計](/2026/n8n-3-pipeline-overview/)
- [個人自動化平台(番外) 拆掉重建 GCP VM & Cloudflared Tunnel & Cloudflare Access](/2026/n8n-3.1-rebuild-infromation/)
- [個人自動化平台(四) n8n 實作：收集層，RSS 資料來源](/2026/n8n-4-pipeline-fetch/)
- [個人自動化平台(五) n8n 實作：處理層，Aggregate + Gemini 週報整理](/2026/n8n-5-pipeline-process/)
- [個人自動化平台(六) n8n 實作：輸出層，Instagram API 取得 Token 全記錄](/2026/n8n-6-pipeline-output/)

## 小結

- cloudflared tunnel 讓 n8n 對外，不用開 GCP 防火牆、不暴露任何 port
- Cloudflare Access 擋在前面，只有通過驗證的 email 才能進到 n8n
- cloudflared 設成 systemd service，VM 重開機自動恢復，不需要手動啟動
- 整條鏈路：瀏覽器 → Cloudflare Access（驗證）→ cloudflared tunnel → n8n

(fin)
