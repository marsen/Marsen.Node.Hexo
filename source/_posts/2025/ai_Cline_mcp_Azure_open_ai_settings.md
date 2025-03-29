---
title: " [實作筆記] 設定 VSCode 外掛 CLine(MCP Client) Azure Open AI 的踩雷筆記"
date: 2025/03/24 17:54:05
tags:
  - 實作筆記
---

## 前情提要

MCP（Model Context Protocol）是一種協議，最近社群很夯的東西，  
所以稍微研究它一下，不論它是否只是個 Buzz word。  

Cline 是開源 AI 程式設計助理，可以外掛至 VS Code，具計劃/執行模式，支援 MCP 協議，  
所以我選擇它當作入門的工具之一。  

## 問題筆記

使用 VSCode 與 Cline 作為起點時，一個雷點就是使用 Azure Open AI 用戶，究竟應該怎麼設定?  
從[此處](https://github.com/search?q=repo%3Acline%2Fcline+OpenAI+Compatible&type=issues)可以看到很多人的討論。  
看來有很多人也常常撞牆，雖然無法全面的測試一輪，
但以下是我對 Azure OpenAI 成功的設置，記錄下來作為以後參考用

API Provider 選擇 `OpenAI Compatible`  
Base URL 我直接從 Azure AI Foundry 上取出:  
在 <https://portal.azure.com/> 找到 Azure AI services | Azure OpenAI，再前往  Azure AI Foundry Portal  
此時你應該可以在 Deployments 找到你的 Targe URI，類似如下  
`https://{resourcename}.openai.azure.com/openai/deployments/{deployment_name}/chat/completions?api-version={apiversion}`  
API Key 就不用多說了，Azure 會提供你兩把金鑰，如果有異常時可以替換它們，  
Model ID 可以在 deployments > Model name 點擊連結後找到，會類似下面這樣  

`azureml://registries/azure-openai/models/gpt-4o/versions/2024-11-20`

## 參考

- <https://www.explainthis.io/zh-hant/ai/mcp>
- <https://cline.bot/>
- <https://www.youtube.com/watch?v=McNRkd5CxFY>
- <https://modelcontextprotocol.io/quickstart/server>
- <https://www.axtonliu.ai/newsletters/ai-2/posts/claude-mcp-protocol-guide>

(fin)
