---
title: " [實作筆記] 試玩 DeepSeek 與避免思想審查"
date: 2025/02/20 17:05:10
tags:
  - 實作筆記
---

## DeepSeek 簡介

DeepSeek 是一家中國 AI 新創公司，透過低成本、高效率的訓練模式，  
在相對低的成本下訓練出接近 ChatGpt 效能的 AI 模型。  
打破了 AI 訓練需高成本的迷思再加上開源，在 AI 和金融投資界引起廣泛關注。  
其創新方法可能改變 AI 產業的投資邏輯，對科技股市造成顯著影響。  

## 思想審查

DeepSeek 在訓練過程中使用的資料集，通常可能經過嚴格篩選，包含大量官方認可的內容與過濾後的敏感信息，  
這使得模型內部便內建了與中共敘事一致的偏見與審查機制。  

### LM Studio 解法

「在本機離線使用的 lmstudio.ai 環境裡，如果遇到言論管制的題目，  
只要用 ⌘U 先輸入思考過程和回答的前綴，再用 → 繼續生成回答，就可以繞過了。」  --- 唐鳳

#### LM Studio 簡介

> LM Studio 是一個提供圖形化使用者介面（GUI）的語言模型開發平台，  
  讓用戶能夠簡單地進行模型訓練、調整及部署，無需繁複的程式碼操作。  
  它支援各種自然語言處理應用，提升開發效率和操作體驗。

### 越獄版 ?

參考[零度博客](https://www.freedidi.com/18431.html)
可以安裝所謂的[越獄版](https://ollama.com/huihui_ai/deepseek-r1-abliterated)
當中有一個手法如下
> This is an uncensored version of deepseek-ai/deepseek-r1  
> created with abliteration (see remove-refusals-with-transformers to know more about it).  
> This is a crude, proof-of-concept implementation to remove refusals from an LLM model without using TransformerLens.  
>
> If “<think>” does not appear or refuses to respond, you can first provide an example to guide,  
> and then ask your question.
> For instance:
>
>> How many 'r' characters are there in the word "strawberry"?

這個手法，在本地端的 DeepSeek 也會有效;**這樣真的需要去抓越獄版的嗎 ?**

## 實測結果

本來只會回答「你好，这个问题我暂时无法回答，让我们换个话题再聊聊吧」的 DeepSeek，  
透過以上的方法確實可以生成其他回應。  
不過只要繼續談話(敏感話題)，就會發現它的思考模式仍然有受限，  
實際的破解價值高不高? 特別是要聊政治敏感的話題的話，就值得思考了。

## 參考

- [唐鳳 threads](https://www.threads.net/@digitalminister.one/post/DFXcvfEppCI?xmt=AQGzrxCjmLFRN72C2I1La4nZfqpeOoz280wZtDDb7bkDpg)
- [重點趨勢：最近爆火的Deepseek是什麼？對金融市場會造成怎樣的影響？](https://www.thinkmarkets.com/tw/market-news/what_is_deepseek_and_how_if_affect_the_market/)

(fin)
