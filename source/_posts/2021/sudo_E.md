---
title: " [A 社筆記] Sudo 與環境變數"
date: 2021/03/17 14:07:29
---

## 前情提要

使用 Azure 的 pipeline 對專案進行 CI/CD，  
在某一段執行語法中，因權限不足無法建立必要的資料夾，  
而導致部署失敗。
有兩個思路，一個是讓現有的使用者擁有建立資料夾的權限，  
另一個想法比較單純，使用 `sudo` 提供足夠的權限給 CI/CD 的執行者。

## 問題

但是 `sudo` 引發了另一個問題，  
設定在 pipeline 的環境變數消失了，  
原因是當我們在使用 `sudo` 的時候，會影響到環境變數。
這個時候需要參考 `/etc/sudoers` 的設定

### env_reset

- env_check # 當變數含有不安全字元 `%` 或 `/` 會被移除
- env_keep # 當 env_reset 為 enable 時要保留的一系列環境變數。
- env_delete # 當 env_reset 為 enable 時要移除的一系列環境變數。

```text
env_reset   If set, sudo will reset the environment to only contain
            the LOGNAME, SHELL, USER, USERNAME and the SUDO_*
            variables.  Any variables in the caller’s environment
            that match the `env_keep` and `env_check` lists are then
            added.  The default contents of the `env_keep` and
            `env_check` lists are displayed when sudo is run by root
            with the -V option.  If the secure_path option is set,
            its value will be used for the PATH environment
            variable.  This flag is on by default.
```

## 解決方法

最簡單的方法加上`-E`, 將 User 的環境變數先載入

```text
The -E (preserve environment) option indicates to the security policy 
that the user wishes to preserve their existing environment variables. 
The security policy may return an error if the -E option is specified 
and the user does not have permission to preserve the environment.
```

## 參考

- [sudo 造成環境變數消失 : sudo -E](https://www.puritys.me/docs-blog/article-392-sudo-%E9%80%A0%E6%88%90%E7%92%B0%E5%A2%83%E8%AE%8A%E6%95%B8%E6%B6%88%E5%A4%B1-:-sudo--E.html)
- [How do I make sudo preserve my environment variables?](https://superuser.com/questions/232231/how-do-i-make-sudo-preserve-my-environment-variables)
- [linux中的sudo許可權](https://www.itread01.com/content/1544986744.html)

(fin)
