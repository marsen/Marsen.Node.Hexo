---
title: "[實作筆記] 配置多語系 i18next "
date: 2021/10/04 09:52:15
---

## 前情提要

多語系是國際化的專案很重要的一部份,  
這次有機會接觸到國際化的前端的案子,  
記錄一下前端主流的 i18next 如何設定。

## 設定

1. 安裝 i18n
  
    ```shell
    # npm
    $ npm install i18next --save

    # yarn
    $ yarn add i18next
    ```

2. 準備多語系 JSON 檔, 為 Key-Value 形式, Value 為 string
   允許巢狀, 範例如下:

    ```json
    {
      "zh": {
        "Week":{
          "Monday":"周一",
          "Tuesday":"周二",
          "Wednesday":"周三",
          "Thursday":"周四",
          "Friday":"周五",
          "Saturday":"周六",
          "Sunday":"周日"
        }
      },
      "es": {
        "Week":{
          "Monday":"Monday",
          "Tuesday":"Tuesday",
          "Wednesday":"Wednesday",
          "Thursday":"Thursday",
          "Friday":"Friday",
          "Saturday":"Saturday",
          "Sunday":"Sunday"
        }
      }
    }
    ```

3. 起始設定

    ```js
    import i18next from 'i18next';
    import resources from './resources'; // 載入上一步的 JSON 檔

    i18next.init({
      lng: 'zh', // 預設語言
      debug: true,
      resources: resources
    });
    
    // 使用方式
    document.getElementById('output').innerHTML = i18next.t('Week.Sunday');
    ```

4. 追加 React 設定

    ```javascript
    import React from "react";
    import i18n from "i18next";
    import { useTranslation, initReactI18next } from "react-i18next";
    import resources from './resources'; // 載入上一步的 JSON 檔

    i18n
      .use(initReactI18next) // passes i18n down to react-i18next
      .init({
        resources: resources,
        lng: "en", // if you're using a language detector, do not define the lng option
        fallbackLng: "en",
        interpolation: {
          escapeValue: false // react already safes from xss => https://www.i18next.com/translation-function/interpolation#unescape
        }
      });
      ```

    在 React 中使用

    ```javascript
        import ReactDOM from "react-dom";
        function App() {
          const { t } = useTranslation();
          return <h2>{t('Week.Sunday')}</h2>;
        }

        // append app to dom
        ReactDOM.render(
          <App />,
          document.getElementById("root")
        );
    ```

其它 [Framework](https://www.i18next.com/overview/supported-frameworks) 請參考

## 參考

- [i18next](https://www.i18next.com/)
- [react-i18next](https://react.i18next.com/)

(fin)
