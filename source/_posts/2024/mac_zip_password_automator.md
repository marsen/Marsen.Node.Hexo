---
title: " [實作筆記] Macbook 壓縮檔案"
date: 2024/01/25 13:17:14å
---

## 前情提要

假設分享機密文件給同事或者發送文件到郵件或雲端儲存時，就必須手動處理壓縮和加密，  
可以參考一些文章，大致有三個作法

- 購買付款壓縮軟體
- 使用雲端服務
- 手動執行 terminal 指令

手動執行就可以處理的問題，我不會特別想要付款買一個軟體，  
而雲端服務會擔心資訊安全，特別是要加密的資料代表有一定程度的重要性。  
Terminal 大概是網路文章的主流解。  
當我需要經常這樣做時，就太麻煩了，還要記得操作指令，額外增加心智負擔。  
理想上，我希望只要選擇文件或資料夾，右鍵點擊"Zip with Password"，就能創建加密的 zip 文件。

## 實作

1. 打開 Automator app。
2. 創建一個新 Automator 文件：File > New (或按 ⌘N)，選擇 "Quick Action" 類型。
3. 將輸入類型更改為 "files or folders"。
4. 在左側的 Actions library 中，雙擊 "Run AppleScript"，或拖放到右側工作區。刪除示例代碼，替換為第 5 步的腳本。
5. 複製並粘貼下面的 AppleScript 代碼。

   ```shell

      set prompt_text to "請輸入壓縮密碼"

      repeat
          set zip_password to text returned of (display dialog prompt_text default answer "" with hidden answer)
          set verify_password to text returned of (display dialog "再次輸入壓縮密碼" buttons {"OK"} default button 1 default answer "" with hidden answer)
          considering case and diacriticals
              if (zip_password = verify_password) then
                exit repeat
              else
                  set prompt_text to "密碼不一致，請重新設定"
              end if
          end considering
      end repeat

      tell application "Finder"
          set the_items to selection
          if ((class of the_items is list) and (count of the_items) > 0) then
              set items_to_zip to ""
              repeat with each_item in the_items
                  set each_item_alias to each_item as alias
                  set item_name to name of each_item_alias
                  set item_name to quoted form of (item_name & "")
                  set items_to_zip to items_to_zip & item_name & " "
              end repeat

              set first_item to (item 1 of the_items) as alias
              set containing_folder to POSIX path of (container of first_item as alias)
              set zip_name to text returned of (display dialog "輸入壓檔名" default answer "")
              set zip_file_name to quoted form of (zip_name & ".zip")

              if zip_password is not equal to "" then
                  -- 如果存在密碼，執行加密壓縮
                  do shell script "cd '" & containing_folder & "'; zip -x .DS_Store -r0 -P '" & zip_password & "' " & zip_file_name & " " & items_to_zip
              else
                  -- 否則執行單純壓縮
                  do shell script "cd '" & containing_folder & "'; zip -x .DS_Store -r0 " & zip_file_name & " " & items_to_zip
              end if
          else
              display dialog "你未選擇任何檔案。" buttons {"OK"} default button 1
          end if
      end tell

   ```

6. 儲存工作流程：File > Save (或按 ⌘S)，給它一個名字，如 "Zip with Password"

現在，選擇文件或文件夾，右擊，選擇 "Quick Actions" 中的 "Zip with Password"，  
按照提示輸入密碼、驗證密碼和設置 zip 文件的名字。測試新創建的 zip 文件，確保一切運作正常。

## 參考

- <https://jimmysie.com/2021/04/30/how-to-create-a-password-protected-zip-file-using-quick-action-on-mac/>
- <https://www.tech-girlz.com/2022/03/mac-password-protect-zip-file.html>
- <https://www.minwt.com/mac/22863.html>
- <https://www.tech-girlz.com/2021/04/mac-zip-with-password.html>

(fin)
