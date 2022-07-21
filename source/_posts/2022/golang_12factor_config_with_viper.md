---
title: "[實作筆記] 12-Factor Config 使用 Golang (with Viper)"
date: 2022/07/19 18:00:08
tag:
  - Golang
---

## 前情提要

最近在開發需要以微服務的形式部署在雲端之上的 web service，
為此，被要求需符合雲原生的 [12 Factory](https://www.12factor.net/)，
在本篇只討論第三點 Config - Store config in the environment,
簡單介紹一下這個因子;

組態應與程式分離
簡單的問自已程式是否可以立即開源，而不擔心洩露任何機敏資料??

先說明一些非微服務的組態設定實踐方式:

### 將組態設定以常數寫在程式當中

> 最糟糕的實踐，你永遠不該在一個產品中這樣作
> 每次改變一個環境，就要修改程式，這會使得你的服務變得脆弱

### 使用不同環境的組態檔，但是不納入版本控制

> 這是個常見的實踐，但是有著以下的缺點
>
> 1. 開發人員將組態檔簽入版控(如果是 Git 應該可以透過 .gitignore 避免這些狀況)
> 2. 配置文件很有可能分散在不同的位置，依照你所使用的語言或框架很有可能會有不同的格式(xml、yaml、json 等等…)  
>    這導致統一管理與維護的困難，特別是在擴展的時候。
>
> > 以前在開發單體服務的時候，一個組態會達上千個設定值，  
> > 而在擴展時，開發者永遠無法理解這上千個組態應該填入什麼值  
> > 這有點超出主題了，但是微服務的架構下，單一服務不應該有上千個組態設定
>
> 有時候我們的環境不止有 `Prod`、`QA`、`Staging`  
> 比如說開發環境、多國/語系/開發人員、壓測，
> 都可能導致環境組態檔案數量爆增，而難以維護。

## 問題 12-Factor 與開發環境

12-Factor 推薦的方式是，使用環境變數來避免上述的問題。  
但在開發環境上這不是很理想的方式

對於新進的開發者:
需要自行加上環境變數，而不是下載程式後即可使用，會形成一種門檻

可以提供 script 或 bash 讓開發者執行後寫入環境變數
但很可能變成一個灰盒子(一次性執行的 script，只有需要編輯時才去會細看)，
而且新增、刪除、修改組態都要修改 script，而產生額外的工作

開發者很可能同時開發多個不同服務，  
這樣如果發生變數的命名衝突會難以解決。

> 生產環境(Prod、QA、Staging)不會有這樣的問題，  
> 因為微服務的一個單體(Pod)應該都是單純的  
> 加上服務名的前綴或許可以解決開發環境的問題，  
> 但是會讓**變數名變得相對冗長**

## 解決方式

1. 提供`development.env`作為開發者的組態檔
2. 程式會優先讀取**環境變數**的值，才會讀取`development.env`的值

Golang 的實作如下:
我們使用 `viper` package，viper 本身包含一定的順序規則去讀取組態，如下:

- explicit call to Set 顯示設定
- flag 命令提示字元參數
- env 環境變數
- config 設定檔
- key/value store 遠端儲存
- default 默認值

程式如下，先讀取設定檔或環境變數的順序並沒有差，  
**因為 Viper 已經提供這樣功能實踐(env 環境變數>config 設定檔)**

```golang=
var instance *Config

func Load() {
  // new viper
  vp := viper.New()
  // just for development environment
  vp.SetConfigName("development.env")
  vp.SetConfigType("env")
  vp.AddConfigPath(".")
  vp.ReadInConfig()
  // follow 12 factors principles https://www.12factor.net/config
  vp.AutomaticEnv()
  // bind to struct
  vp.Unmarshal(&instance)
}
```

`struct` 與 `development.env` ，純量或是巢狀結構都可以支援，  
見以下的程式 `Config` 內部包含另一個 struct `NestConfig`

```go=
type Config struct {
  Port string     `env:"PORT"`
  Nest NestConfig `env:"NestConfig"`
}

type NestConfig struct {
  MarsenNo   int  `env:"MarsenNo"`
  MarsenFlag bool `env:"MarsenFlag"`
}
```

`development.env`範例如下

```env=
Port=9009
Nest.MarsenNo=1231456
Nest.MarsenFlag=1
```

Config 的使用：
這裡用了 singleton 的方法實作，有查看了一些[文章](https://blog.kennycoder.io/2021/08/22/Golang-Singleton-%E5%AF%A6%E7%8F%BE%E6%96%B9%E5%BC%8F%E6%8E%A2%E8%A8%8E/)可能會有問題，未來可能會再調整

```go=
var instance *Config

func Of() *Config {
  if instance == nil {
    Load()
  }
  return instance
}
```

**[完整代碼可以見此](https://gist.github.com/marsen/592d7aa1da912beffaad1e1f0c47b086#file-config-go)**

### 未解之問題

提供給開發者使用的組態，仍有機敏資料進版的疑慮?  
考慮使用 [Consul](https://www.consul.io/) ???  
或是改成 docker image 並在其中寫入環境變數 ???

## 參考

- [III. Config - Store config in the environment](https://www.12factor.net/config)
- [Viper](https://github.com/spf13/viper#why-viper)
  - [GO 12factor Example (Web Service)](https://github.com/utain/go-12factor-example)
  - [Golang Viper Config Example](https://github.com/devilsray/golang-viper-config-example)
  - [在 Go 語言使用 Viper 管理設定檔](https://blog.wu-boy.com/2017/10/go-configuration-with-viper/)
  - [Handling Go configuration with Viper](https://blog.logrocket.com/handling-go-configuration-viper/)
  - [Load config from file & environment variables in Golang with Viper](https://dev.to/techschoolguru/load-config-from-file-environment-variables-in-golang-with-viper-2j2d)
  - [Go Application Configuration with Viper | Config with JSON](https://www.youtube.com/watch?v=r9Qvr40H4eE)
  - [Golang 的配置信息處理框架 Viper](https://www.readfog.com/a/1632554002629627904)
- [Golang - Singleton 實現方式探討](https://blog.kennycoder.io/2021/08/22/Golang-Singleton-%E5%AF%A6%E7%8F%BE%E6%96%B9%E5%BC%8F%E6%8E%A2%E8%A8%8E/)

(fin)
