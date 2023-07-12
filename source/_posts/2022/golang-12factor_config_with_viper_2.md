---
title: " [實作筆記] 12-Factor Config 使用 Golang (with Viper) II"
date: 2022/08/09 11:56:00
tags:
  - 實作筆記
  - Golang
---

## 前情提要

在前篇提到了如何透過 Viper 來取得 config 檔與環境變數來組成我們的組態檔。
這篇我們會進一步討論，透過 Go 的強型別特性，取得組態物件，並處理巢狀的結構。

## 假想情境

假設今天我們要作一個金流服務，  
後面可能會介接 PayPal、Strip 等等不同金流的服務，
那我的組態設定上可能就會有不同的階層關係

```json
{
  "Version": "1.10.1",
  "Stripe": {
    "Enable": true,
    "MerchantId": 123,
    "ClientId": 123
  },
  "Paypal": {
    "Enable": false,
    "MerchantNo": "A003"
  }
}
```

那麼問題來了

1. 我應該怎麼取用這些組態 ?
   - 可以直接透過 `viper.Get("name")` 取得值
   - 但是每次都要用組態名稱字串取得值，其實是一個風險;
     比如打錯字，很難有工具有效防範
   - 所以我希望建立一個 type 去定義組態檔，並透過物件去存取，
     這可以有效將寫死字串限定至一次。
2. 有了上述的方向後，另一個問題是巢狀解構的解析，
   一般來說，我認為組態檔不應該有超過三層的深度，這次就用兩作為範例說明
3. Viper 本身支援多種組態格式，本篇僅以 `env`、`json`、`yaml` 作範例說明
4. 在上一篇提到，當存在環境變數時，環境變數的優先度應高於檔案

## 期待的結果

我們在啟動程式的時候，一次性將組態載入事先定義好的物件之中，  
如果有環境變數，優先取用。  
如果沒有環境變數，會讀開發者提供的組態檔(env 檔優先，也要能支援 json 或 yaml )  
如果完全沒有組態，應該發出警告

## 定義 Type

首先建立我們的 Type

```golang
type Config struct {
  Version string
  Stripe  StripeType
  Paypal  PaypalType
}

type StripeType struct {
  Number int
  Enable bool
}

type PaypalType struct {
  MerchantNo string
  Enable     bool
}
```

接下來我們如果上一篇宣告 `Of` 方法用來取得組態

```golang
// define instance
var instance *Config
func Of() *Config {
  once.Do(func() {
    Load()
    })
  return instance
}
```

此時我們要將組態載入，先考慮開發者環境的實體檔案

```golang
func Load() {
  vp := viper.New()
  vp.AddConfigPath(".")
  // todo here, change config name & type
  vp.SetConfigName("config.json")
  vp.SetConfigType("json")
  vp.ReadInConfig()
  vp.Unmarshal(&instance)
}
```

### 實作 env 使用的 config.env 檔案如下

```env
Version="1.10.1.yaml"
Stripe.Enable=true
Stripe.Number=123
Paypal.Enable=false
Paypal.Merchant_No="A003"
```

修改 viper 相關設定

```golang
// just for development environment
vp.SetConfigName("config.env")
vp.SetConfigType("env")
```

### 實作 yml 使用的 config.yml 檔案如下

```yml
Version: "1.10.1.yml"
Stripe:
  Enable: true
  Number: 765
Paypal:
  Enable: true
  MerchantNo: "Yml003"
```

修改 viper 相關設定

```golang
// just for development environment
vp.SetConfigName("config.yml")
vp.SetConfigType("yml")
```

### 實作 json 使用的 config.json 檔案如下

```json
{
  "version": "1.10.1.json",
  "stripe": {
    "enable": true,
    "number": 123
  },
  "paypal": {
    "enable": true,
    "merchantNo": "Json003"
  }
}
```

修改 viper

```golang
// just for development environment
vp.SetConfigName("config.json")
  vp.SetConfigType("json")
```

以上提供了幾種不同的 type

## 環境變數

我使用的環境變數如下

```env
STRIPE__ENABLE=true;STRIPE__No=678;VERSION=8.88.8;PAYPAL__ENABLE=false;PAYPAL__MERCHANT_NO=ENV9999
```

首先，我們知道環境變數並不像 `json` 或 `yaml` 檔一樣可以提供巢狀的結構，
這就與我們的需求有了衝突，好在 `viper` 提供了 `BindEnv` 的方法, 我們可以強制讓它建立出巢狀的結構，
如下:

```golang
vp.BindEnv("VERSION")
vp.BindEnv("Stripe.NUMBER", "STRIPE__No")
vp.BindEnv("Stripe.Enable", "STRIPE__ENABLE")
vp.BindEnv("Paypal.Enable", "PAYPAL__ENABLE")
vp.BindEnv("Paypal.MerchantNo", "PAYPAL__MERCHANT_NO")
```

寫在後面，在查找資料的過程中，可以發現 viper 提供了兩個功能強大的方法，
SetEnvPrefix 與 SetEnvKeyReplacer ;
SetEnvPrefix 可以自動加上前綴，SetEnvKeyReplacer 可以將分隔符置換。
可惜在我的情境尚且用不到，未來再作研究。

## 參考

- [Tutorial Series: How To Code in Go](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go)
- [Go 语言中的 struct tag](https://zhuanlan.zhihu.com/p/32279896)
- [Configuring JSON struct fields](https://riptutorial.com/go/example/14157/configuring-json-struct-fields)
- [在 Go 語言使用 Viper 管理設定檔](https://blog.wu-boy.com/2017/10/go-configuration-with-viper/)

(fin)
