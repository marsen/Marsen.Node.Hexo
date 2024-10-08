---
title: " [學習筆記] TypeScript tsconfig 設定備忘錄"
date: 2024/10/08 23:28:35
tags:
  - 學習筆記
---

## 前情提要

tsconfig.json 文件是一個 TypeScript 專案的配置文件，它位於根目錄中，並定義了專案的編譯器選項。
提供給開發人員輕鬆配置TypeScript 編譯器，並確保專案的程式碼在不同的環境中始終保持一致。
你可以使用 `tsc --init` 指令自動產生。
也可以參考 Matt Pocock 的[TSConfig Cheat Sheet](https://www.totaltypescript.com/tsconfig-cheat-sheet)

TSConfig 有上百個配置，本文將用來重點記錄一些相關的配置

## 本文

### 2024/10 Matt Pocock 的建議配置

相關說明請參考[原文](https://www.totaltypescript.com/tsconfig-cheat-sheet)

```json
{
  "compilerOptions": {
    /* Base Options: */
    "esModuleInterop": true,
    "skipLibCheck": true,
    "target": "es2022",
    "allowJs": true,
    "resolveJsonModule": true,
    "moduleDetection": "force",
    "isolatedModules": true,
    "verbatimModuleSyntax": true,

    /* Strictness */
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,

    /* If transpiling with TypeScript: */
    "module": "NodeNext",
    "outDir": "dist",
    "sourceMap": true,

    /* AND if you're building for a library: */
    "declaration": true,

    /* AND if you're building for a library in a monorepo: */
    "composite": true,
    "declarationMap": true,

    /* If NOT transpiling with TypeScript: */
    "module": "preserve",
    "noEmit": true,

    /* If your code runs in the DOM: */
    "lib": ["es2022", "dom", "dom.iterable"],

    /* If your code doesn't run in the DOM: */
    "lib": ["es2022"]
  }
}
```

### files

預設值為 `false`，如果只有少量的 ts 檔，很適合設定這個`files`
但實務上更常用 `include` 與 `exclude` 搭配，  
兩者都可使用 wildcard ，exclude 擁有較高的優先級，
例如下面的設定 `src/.test` 底下的檔案將不會被編譯:

```json
{
  "compilerOptions": {},
  "include": ["src/**/*"],
  "exclude": ["src/test/**/*"]
  "files": [
    "a_typescript_file.ts",
    "other_ts_code.ts"
  ]
}
```

### Compiler Options

這部份是 tsconfig 相關設定的主體，有需要時來[這裡](https://www.typescriptlang.org/tsconfig/#compilerOptions)查就好
下面的例子是一些常用的設定說明:

```json
{
  "compilerOptions": {
    // esModuleInterop: 這個選項使 TypeScript 能夠更好地與 CommonJS 模組兼容，允許以 ES 模組方式導入 CommonJS 模組。
    "esModuleInterop": true,
    // skipLibCheck: 當設置為 true 時，TypeScript 將跳過對聲明檔案（.d.ts）的型別檢查，可以加快編譯速度，尤其是在大型專案中。
    "skipLibCheck": true,
    // target: 指定編譯後的 JavaScript 代碼的 ECMAScript 版本。這裡設定為 ES2022，這意味著生成的代碼將使用該版本的新特性。
    "target": "es2022",
    // allowJs: 此選項允許在 TypeScript 專案中使用 JavaScript 檔案，這對於逐步遷移到 TypeScript 的專案特別有用。
    "allowJs": true,
    // resolveJsonModule: 使 TypeScript 能夠導入 JSON 檔案，並將其視為模組。
    "resolveJsonModule": true,
    // moduleDetection: 設置為 force 使 TypeScript 將所有檔案視為模組，避免了使用全局變數引起的錯誤。
    "moduleDetection": "force",
    // isolatedModules: 每個檔案將被獨立編譯，這對於使用 Babel 或其他工具的場景特別重要。
    "isolatedModules": true,
    // verbatimModuleSyntax: 強制使用類型專用的導入和導出，使 TypeScript 更加嚴格，這樣有助於在編譯時優化生成的代碼。
    "verbatimModuleSyntax": true
  }
}
```

### Watch Option

`watchOptions` 用於配置 TypeScript 在開發過程中如何監控文件變更。  
這些選項主要用於改善開發效率，當監控的文件或目錄發生變化時，TypeScript 會自動重新編譯或執行其他指定操作。  
這對於開發階段非常重要，因為它允許**開發者即時查看更改的影響，而不必手動重新編譯代碼。**

```json
{
  "compilerOptions": {},
  "watchOptions": {
    // watchFile: 用於設定 TypeScript 監控單個檔案的方式。
    // fixedPollingInterval: 每隔固定時間檢查所有檔案是否變更。
    // priorityPollingInterval: 根據啟發式方法，對某些類型的檔案進行優先輪詢。
    // dynamicPriorityPolling: 使用動態隊列，較少修改的檔案將會被較少檢查。
    // useFsEvents (預設): 嘗試使用操作系統/檔案系統的原生事件來監控檔案變更。
    // useFsEventsOnParentDirectory: 嘗試監聽檔案父目錄的變更事件，而不是直接監控檔案。
    "watchFile": "useFsEvents", // 預設為使用文件系統事件來監控檔案變更。
    
    // watchDirectory: 用於設定 TypeScript 監控整個目錄的方式。
    // fixedPollingInterval: 每隔固定時間檢查所有目錄是否變更。
    // dynamicPriorityPolling: 使用動態隊列，較少修改的目錄將會被較少檢查。
    // useFsEvents (預設): 嘗試使用操作系統/檔案系統的原生事件來監控目錄變更。
    "watchDirectory": "dynamicPriorityPolling", // 使用動態優先級輪詢，檢查變更較少的目錄次數較少。
    
    // 設置檔案或目錄的輪詢間隔時間（以毫秒為單位），適用於輪詢策略。
    "pollingInterval": 2000, // 設置為每 2000 毫秒輪詢一次檔案或目錄變更。
    
    // 設置是否監控目錄及其所有子目錄
    "recursive": true // 設置為 true 以監控所有子目錄的變更。
  }
}
```

### Type Acquisition

Type Acquisition 主要適用於 JavaScript 專案。  
在 TypeScript 專案中，開發者必須明確地將型別包含在專案中。  
相對地，對於 JavaScript 專案，TypeScript 工具會自動在後台下載所需的型別，並將其儲存在 node_modules 以外的位置。

```json
{
  "compilerOptions": {},
  "typeAcquisition": {
    // enable: 啟用自動獲取 JavaScript 專案的型別檔案 (.d.ts)。
    "enable": true,

    // include: 指定自動獲取型別檔案的庫。
    // 例如，在這裡指定了 jQuery 和 lodash，TypeScript 將自動獲取它們的型別檔案。
    "include": ["jquery", "lodash"],

    // exclude: 指定不需要自動獲取型別檔案的庫。
    // 這裡指定了 "some-legacy-library"，即使它存在於專案中，TypeScript 也不會嘗試獲取它的型別檔案。
    "exclude": ["some-legacy-library"],

    // disableFilenameBasedTypeAcquisition: 禁用基於檔案名稱自動獲取型別檔案的功能。
    // 當設置為 true 時，TypeScript 不會基於檔案名稱來推測並下載型別檔案。
    "disableFilenameBasedTypeAcquisition": false
  },
}
```

## 參考

- [Intro to the TSConfig](https://www.typescriptlang.org/tsconfig/)
- [The TSConfig Cheat Sheet](https://www.totaltypescript.com/tsconfig-cheat-sheet)
- [What is a tsconfig.json](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html#handbook-content)

(fin)
