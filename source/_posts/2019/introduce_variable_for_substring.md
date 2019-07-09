---
title: "[學習筆記]重構使用 Introduce Variable for Substring "
date: 2019/07/09 20:02:13
---

## 要知道的事

- 這是個人的學習記錄
- 可能對你沒幫助
- 不知不知最可怕。快不起來,因為你不覺得慢
- 希望對你有幫助
- 使用 Visual Studio & ReSharper

## 問題

> 把原本的程式
> string url = "http://localhost:5000/api/v3.3/refund/PayPal/SF188964T"
>
> 變成下面這樣
>
> string domain = "http://localhost:5000";  
> string code = "SF188964T";  
> string payType = "PayPal";  
> string url = $"{domain}/api/refund/{payType}/{code}";  
>
> 你會怎麼作 ?

我原本的作法，一個字一個挖出來，命名變數，再填回去。
怎麼作可以更快 ?

透過 Resharper 的 Refactor > Introduce Variable > Introduce Variable for Substring 即可快速重構。

參考 91 的影片
{% youtube Nqz8qapjjz4 %}

我實測原本的開發時間約為2~3分鐘，善用工具約可以在 30 ~ 45 s 完成。
效能提昇 400 % !!!!!(聽起來就很威)

{% youtube C9jLXrh3LRg %}

當然結合了 Vim 與 Visual Studio 的工具才達到這樣的速度，強烈推薦以下課程。

## 我只推薦好東西

[【極速開發+】 202002 第九梯次 台北](https://dotblogs.com.tw/hatelove/2019/06/17/extreme-developing-training-202002)

### 課程優點

1. 學心法也學作法，不是只有理論的打高空
2. 認識同好 ，來上課的同學都是願意精進自已的人，所以身上有很多寶可以挖
3. 認識 91 ，遇到 91 盡量挖就對了，偷到一招半式都完勝 10 年在那邊處理 NullReferenceException
4. 學習怎麼學習

## 其它本周學習事項

1. 可以讓中斷點停留在 lambda express 中
2. MSTest 請用 DescriptionAttribute 加入測試描述
3. [.Net MVC Model Binding 若欄位為空字串預設為轉為 Null](https://docs.microsoft.com/zh-tw/dotnet/api/system.web.ui.webcontrols.boundfield.convertemptystringtonull?view=netframework-4.8)

## 本周待確認事項

1. 如何對 HttpClient 測試 ?
2. 如何用 [FluentValidation](https://fluentvalidation.net/testing) 作測試 ?

(fin)
