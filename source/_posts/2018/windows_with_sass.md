---
title: " [實作筆記] 在 Windows 環境編輯 SCSS"
date: 2018/04/26 14:16:34
tag:
  - Windows
  - SCSS
  - Ruby
  - Nodejs
  - 實作筆記
---

1. 安裝 Windows 套件管理工具 Chocolatey
   - <https://chocolatey.org>
2. 安裝 Node.js®

   - <https://chocolatey.org/packages/nodejs>

   ```sh
   choco install nodejs -y
   ```

3. 註冊公司內部 NPM Server

   - <http://company.npm.server>

   ```sh
   npm set registry http://company.npm.server
   ```

4. 安裝 RUBY

   - <https://chocolatey.org/packages/ruby/2.5.1.1>

   ```sh
   choco install ruby -y
   ```

5. 下載 RubyGems
   - <https://rubygems.org/pages/download#formats>
   - Download from above
   - Unpack into a directory and cd there
   - Install with: ruby setup.rb (you may need admin/root privilege)
6. 安裝 compass

   - <https://rubygems.org/gems/compass/versions/1.0.3>

   ```sh
   gem install compass
   ```

7. 檢查 PATH
8. 重啟 CMD 與 Visual Studio 2017
9. 執行 compass

   - 使用 Command Line

     ```sh
     gulp compass
     ```

   - 使用工作執行器總管

     ![工作執行器總管](https://i.imgur.com/2sEzAx5.jpg)

(fin)
