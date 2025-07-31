---
title: " [實作筆記] AI Agent 實作 Web Search API：從設計到部署的完整記錄"
date: 2025/07/31 16:51:10
tags:
  - 實作筆記
---

## 前情提要

最近購買 Claude Code，在此同時也有試用 Github Copilot / Gitlab Duo / Gemini / Amazon Q

剛好手上有需求，是簡單的 API 串接，但是我們的系統架構有一些只有團隊知道的 Know How。

我想試試用 AI Agent 來協助我處理這些開發，需求簡單明確，但是技術細節並不少，

如果是新進 RD(即使有開發經驗)也不見得能掌握得很好，我來試試看 AI Agent 能作什麼程度。

這篇文章記錄我在基於 Clean Architecture 的 Node.js API 系統中實作 Web Search 功能的完整過程，包含架構設計、權限管理、錯誤處理等細節。

## 系統架構背景

專案採用 **Clean Architecture** 搭配 **依賴注入 (Inversify)**，技術棧包含：

- TypeScript + Express.js
- TypeORM (資料庫 ORM)
- Inversify (依賴注入容器)
- JWT (身份驗證)
- Zod (參數驗證)

架構分為四個主要層級：

```text
src/
├── adapters/          # 外部介面層 - 控制器
├── domain/           # 領域層 - 核心業務邏輯  
├── infrastructure/   # 基礎設施層 - 外部依賴實作  
└── useCases/        # 應用層 - 業務用例
```

## 需求分析與設計決策

### API 規格定義

- **端點**: `GET /api/v1/websearch?keyword=關鍵字`
- **輸入**: Query 參數 `{ keyword: string }`
- **輸出**: `{ data: [{ title: string, url: string, description: string }] }`
- **結果數**: 10 筆
- **權限**: 所有 API 都需要權限檢查(我們之前開發好的 RBAC 權限系統 )，一個功能對應一個權限，為此我需要新增 `web_search` 權限

### 關鍵設計決策

**1. 控制器選擇**  

決定將功能放在 `BasicController` 而非 `ProjController`，因為這是通用功能而非特定業務邏輯。

| 這裡我在需求上有明確告知 AI，實作上也沒有問題

**2. 依賴關係**  
嚴格遵循 Clean Architecture 的依賴規則：

| 也有寫入 CLAUDE.md 的開發準則中，但 AI 會常常忘記這件事

```text
Controller → UseCase → Service → External API
```

**3. 環境變數策略**  

將 Google API 設定為非必要欄位，未設定時警告但不中斷其他服務。

| AI 提供很好的建議並快速完成開發

**4. 錯誤處理設計**  

- 503: API 未設定
- 502: API 呼叫失敗  
- 400: 參數錯誤 (Zod 驗證)

## 實作步驟詳解

### Phase 1: 環境設定擴充

首先擴充環境變數設定，新增 Google Search API 相關設定：

```bash
# .env.example
# Google Search API Configuration
GOOGLE_API_KEY=your_google_api_key
GOOGLE_SEARCH_ENGINE_ID=your_search_engine_id  
GOOGLE_SEARCH_API_URL=https://www.googleapis.com/customsearch/v1
GOOGLE_SEARCH_RESULTS_COUNT=10
```

**修改 envConfigService.ts**  
新增 `getOptional()` 方法支援可選配置：

```typescript
public getOptional(key: string): string | undefined {
  return process.env[key]
}
```

### Phase 2: 權限系統整合

建立資料庫遷移檔案，新增 `web_search` 權限：

```sql
INSERT INTO permissions (name, description) VALUES 
('web_search', 'Web Search API access permission');

-- 分配給 admin 和 member 角色
INSERT INTO role_permissions (role_id, permission_id) 
SELECT r.id, p.id FROM roles r, permissions p 
WHERE r.name IN ('admin', 'member') AND p.name = 'web_search';
```

### Phase 3: 核心架構實作

#### 1. 定義領域介面

**src/domain/interfaces/webSearchService.ts**

```typescript
export interface WebSearchService {
  search(keyword: string): Promise<WebSearchResult[]>
}

export interface WebSearchResult {
  title: string
  url: string  
  description: string
}
```

#### 2. Google Search Service 實作

**src/infrastructure/services/googleSearchService.ts**

```typescript
@injectable()
export class GoogleSearchService implements WebSearchService {
  constructor(
    @inject(TYPES.HttpClient) private readonly httpClient: IHttpClient,
    @inject(TYPES.EnvConfigService) private readonly envConfig: IEnvConfigService
  ) {}

  async search(keyword: string): Promise<WebSearchResult[]> {
    const apiKey = this.envConfig.getOptional('GOOGLE_API_KEY')
    const searchEngineId = this.envConfig.getOptional('GOOGLE_SEARCH_ENGINE_ID')
    
    if (!apiKey || !searchEngineId) {
      throw new AppError('Google Search API 未設定', 503)
    }

    const params = {
      key: apiKey,
      cx: searchEngineId, 
      q: keyword,
      num: this.envConfig.get('GOOGLE_SEARCH_RESULTS_COUNT', '10')
    }

    try {
      const response = await this.httpClient.get(
        this.envConfig.get('GOOGLE_SEARCH_API_URL'), 
        { params }
      )
      return this.transformGoogleResponse(response.data)
    } catch (error) {
      throw new AppError('搜尋服務暫時無法使用', 502)
    }
  }

  private transformGoogleResponse(data: any): WebSearchResult[] {
    if (!data.items) return []
    
    return data.items.map((item: any) => ({
      title: item.title,
      url: item.link,
      description: item.snippet || ''
    }))
  }
}
```

#### 3. UseCase 層實作

**src/useCases/ExecuteWebSearch.ts**

```typescript
@injectable()
export class ExecuteWebSearch implements IUseCase<ExecuteWebSearchDTO, WebSearchResponseDTO> {
  constructor(
    @inject(TYPES.WebSearchService) private readonly webSearchService: WebSearchService
  ) {}

  async execute(input: ExecuteWebSearchDTO): Promise<WebSearchResponseDTO> {
    const results = await this.webSearchService.search(input.keyword)
    return { data: results }
  }
}

// DTO 定義
export interface ExecuteWebSearchDTO {
  keyword: string
}

export interface WebSearchResponseDTO {
  data: WebSearchResult[]
}
```

#### 4. Controller 層實作

**src/adapters/controllers/basicController.ts**

```typescript
import { z } from 'zod'

// Zod 驗證 schema
const webSearchQuerySchema = z.object({
  keyword: z.string().min(1, 'keyword 參數為必填且不能為空字串')
})

async webSearch(req: Request, res: Response): Promise<void> {
  // 使用 Zod 進行參數驗證
  const { keyword } = webSearchQuerySchema.parse(req.query)
  
  const input: ExecuteWebSearchDTO = { keyword }
  const result = await this.executeWebSearch.execute(input)
  
  res.status(200).json(result)
}
```

#### 5. 路由設定

**src/infrastructure/routes/basicRouter.ts**

```typescript
// 註冊路由，先執行權限檢查，再執行業務邏輯
r.get(`${basePath}/websearch`, 
  auth.user('web_search'),  // 權限中介軟體
  asyncWrapper(basicController.webSearch.bind(basicController))
)
```

### Phase 4: 依賴注入設定

```typescript
// inversify.config.ts
// 註冊 UseCase
container.bind<IUseCase<ExecuteWebSearchDTO, WebSearchResponseDTO>>(
  TYPES.ExecuteWebSearch
).to(ExecuteWebSearch)

// 註冊 Service
container.bind<WebSearchService>(
  TYPES.WebSearchService
).to(GoogleSearchService)

// 更新 TYPES 常數
export const TYPES = {
  // ... existing types
  ExecuteWebSearch: Symbol.for('ExecuteWebSearch'),
  WebSearchService: Symbol.for('WebSearchService'),
}
```

## 測試與驗證

### 準備測試環境 -- 取得測試 Token

```bash
curl -X 'POST' 'http://localhost:4578/api/v1/auth/login' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{"account": "test_act", "password": "test_pwd"}'
```

### 測試案例設計

| AI　會自動幫我跑 End To End 測試（本來是 RD 用curl 或 postman 進行的工作）
| 案例的設計上也很細心，很多 RD 是只測試 Happy Case 的

#### 正常功能測試(happy case)

```bash
curl -X GET "http://localhost:4578/api/v1/websearch?keyword=nodejs" \
  -H "Authorization: Bearer <token>"
# 預期: Google 搜尋結果 JSON
```

#### 權限檢查失敗測試

```bash
curl -X GET "http://localhost:4578/api/v1/websearch?keyword=test"
# 預期: {"error":"Authentication invalid"}
```

#### 參數驗證測試

```bash
# 缺少參數
curl -X GET "http://localhost:4578/api/v1/websearch" \
  -H "Authorization: Bearer <token>"
# 預期: Zod 驗證錯誤，400 status

# 空參數
curl -X GET "http://localhost:4578/api/v1/websearch?keyword=" \
  -H "Authorization: Bearer <token>"
# 預期: {"error":[{...,"message":"keyword 參數為必填且不能為空字串"}]}
```

## 錯誤處理機制

### Zod 驗證整合

> 專案已內建 Zod 在 req/res 檢查參數與錯誤處理有很好的表現，是團隊的開發工具之一  
> 但要提醒 AI 不然他會手刻一個錯誤處理給你（刻得也不差就是了）

```typescript
// errorHandler.ts 已支援
{
  type: ZodError,
  status: 400,
  log: 'Zod validation error',
  getMsg: (err: ZodError) => err.errors,
}
```

#### ❌ 錯誤方式：AI 手刻錯誤處理

```typescript
if (!keyword || typeof keyword !== 'string') {
  res.status(400).json({ error: 'keyword 參數為必填且必須是字串' })
  return
}
```

#### ✅ 正確方式：使用 Zod  

```typescript
const { keyword } = webSearchQuerySchema.parse(req.query)
```

使用 Zod 的好處：

- 統一的錯誤格式
- 自動整合到全域錯誤處理器
- 型別安全保證

### 中介軟體執行順序

> 重構算是這次需求的大目標，只要提示詞寫得夠好 AI 可以提供很完整的路由列表
> 而且重構的狀態也很正確，不過我的專案不大只有 30 隻左右的 API 參加價值可能不高

```text
請求 → auth.user('web_search') → asyncWrapper → zod.parse() → 業務邏輯
```

這個順序確保：

1. 先檢查身份權限
2. 再進行參數驗證  
3. 最後執行業務邏輯

## Google Custom Search API 整合要點

> 商業需求的主邏輯，我一行程式沒寫只與 AI 互動就完成了這個功能，包含點對點的測試  
> 不過在零信任原則下，還是請其他 RD 再作一次 Code Review 與完整測試

### 必要參數說明

- `key`: Google API Key
- `cx`: Custom Search Engine ID  
- `q`: 搜尋關鍵字
- `num`: 結果數量

### API 回應轉換

Google API 回應結構：

```json
{
  "items": [
    {
      "title": "搜尋結果標題",
      "link": "https://example.com",
      "snippet": "搜尋結果摘要"
    }
  ]
}
```

轉換為系統格式：

```json
{
  "data": [
    {
      "title": "搜尋結果標題",
      "url": "https://example.com", 
      "description": "搜尋結果摘要"
    }
  ]
}
```

## 小結

這次實作讓我深度體驗了 AI 與優良架構在實際專案中的運用。幾個關鍵收穫：

### 架構優勢

- 清晰的分層讓職責分明，測試容易
- 依賴注入讓元件可抽換，符合開放封閉原則
- 統一的錯誤處理機制，維護成本低

上面是原本的優勢，加上 AI 判讀後，可以高效產生 Clean Code  

不太確定不良代碼會有什麼結果，很幸運是我不在那種環境之中  

現在三個圈圈是有交集的，好又快又便宜(產出／單位時間)，  

依照 AI 帶來的生產力與現有的 RD 相比，其實是很便宜的選擇。

良好的設計結合工具(AI)是提昇效率的手段,  

3 個人可以當 ７ 個人用。至於為什麼是３個人，有機會再說了。

(fin)
