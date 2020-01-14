---
title: "[實作筆記] Reshaper Code Template"
date: 2020/01/08 17:17:17
---

## 前情提要

我通常會作一些 Code Snippet 來加速開發，  
避免同樣的代碼要重複的寫，  
除了避免重複，能交給工具的儘量交給工具去作，  
只要是人為的操作就有可能犯錯，就算是簡單的剪下貼上，
讓工具作事，用心去檢驗，這是我在 2019 練習的小心得。  

比如說我使用無蝦米輸入法、刻意練習英打速度，  
購買 Reshaper 等…，都是為了提昇生產力。  
今天就是要說說 Reshaper 的 Template ，  
這是一個跟 Code Snippet 相同的功能 。  

## 本文

Template 與 Code Snippet 並不使用共同的範例檔，
所以要如何將已有的 Snippet 轉移過來呢 ? 如果有人知道請跟我說。
另一個問題是，他真的比原本的 Snippet 好用嗎 ?

因為我的 Snippet 並不多，大約 10 個左右
為了馬上增加生產力，我手動將主要有用到的 2個重建了
剩下的如果真的很少用，就讓他自然淘汰吧。
常用的如果有用到再來重建。

第二件事，我使用上手感是差不多的
據說 Resharper 會更智慧化，所以我會選用 Code Template  

## 如何新增

以 Visual Studio 2019 與 Resharper Ultimate 為例，  
在搜尋框輸入 `Template` 找到 `Template Explorer`  
點擊新增圖示，進行編輯，
其它就跟原本的 Snippet 差不多，也可以提供參數化入與 short cut
![Auto Scaling Group](/images/2020/1/reshaper_tempalte_01.jpg)

## 補充

如果已經安裝了 Resharper Ultimate 但是 Template 不工作的話記得看一下設定

Resharper > Option > Intellisense > General

![Auto Scaling Group](/images/2020/1/reshaper_tempalte_02.jpg)

## 參考

- <https://blog.darkthread.net/blog/vs-code-snippet-with-resharper/>
- <https://www.jetbrains.com/help/resharper/Creating_a_File_Template.html>

(fin)
