---
title: "[實作筆記] Unit Testing With TypeScript"
date: 2019/05/16 13:18:16
tag:
    - Unit Testing
    - TDD
    - TypeScript
    - Node.js
---

## 前情提要

- 難得有機會寫前端的東西
- 其實只有寫 JavaScript
- 我要用 TypeScript 寫
- 合理的軟體工序 TDD
- 所以我要寫測試
- node 版本 v8.11.1
- npm 版本 5.6.0
- typescript 版本 3.3.3333

## Context & User Stories  

來自前端的需求，在一個日曆工具要加入對可選日期判斷的邏輯。  
原始需求如下，g、h、i、j 可以透過修改日曆元件選項調整，  
而細微的日期與時間判斷需要撰寫新的方法作判斷。  

### 原始需求

> a. 每週一過中午12點不能選週二及以前的日期  
> b. 每週二過中午12點不能選週三及以前的日期  
> c. 每週三過中午12點不能選週四及以前的日期  
> d. 每週四過中午12點不能選週五及以前的日期  
> e. 每週五過中午12點不能選隔週一及以前的日期  
> f. 每週日都不能選  
> g. ~~90天以後的日期不能選~~  
> h. ~~需指導我們如何讓特定日期不能選，以因應遇到國定假日的狀況~~  
> i. ~~預設為選擇最近一個可以使用的日期~~  
> j. ~~改成中文~~  




## 測試環境準備

### 使用 Mocha 與 Chai

```shell
npm i -D chai mocha nyc ts-node typescript
```

#### 安裝 [TypeScript](https://www.typescriptlang.org)
安裝至專案

```shell
$ npm i -D typescript ts-node
```

安裝至全域

```shell
$ npm i -g typescript
```
建立專案 `tsconfig.json`
```
$ tsc --init
```

#### 安裝 [MochaJs](https://mochajs.org/)

```shell
npm i -D mocha @types/mocha
```

#### 安裝 [Chai](https://www.chaijs.com/)
```shell
npm i -D Chai @types/chai
```

#### 設定測試
package.json\
```script
"scripts":{
    "test": "mocha -r ts-node/register tests/**/*.test.ts",
}
```

#### 寫測試

```script
import { expect } from 'chai';
import Calculator from '../src/calculate';

describe('calculate', function() {
  it('add', function() {
    let result = Calculator.Sum(5, 2);
    expect(result).equal(7);
  }); 
});
```

#### 實作代碼
```script
export default class calculator {
    static Sum(a: number, b: number): number {
  
    }
```

#### 執行測試
```shell
npm t
```

#### 修正代碼
```script
export default class calculator {
    static Sum(a: number, b: number): number {
        let c = a + b;
        eturn c;
    }
```

#### 測試結果

```shell
  calculate
    √ add


  1 passing (61ms)
```

## 測試覆蓋率
// TODO

### 安裝 nyc

```shell
npm i -D nyc
```

### 設定 scripts

```script
"scripts":{    
     "testCover": "nyc -r lcov -e .ts -x \"*.test.ts\" mocha -r ts-node/register tests/**/*.test.ts && nyc report",
}
```

### 執行 

```shell
> npm run testCover
```

### 執行結果

```shell
略過測試部份
------------------------|----------|----------|----------|----------|-------------------|
File                    |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
------------------------|----------|----------|----------|----------|-------------------|
All files               |      100 |      100 |    98.63 |      100 |                   |
 src                    |      100 |      100 |     87.5 |      100 |                   |
  Calculator.ts         |      100 |      100 |      100 |      100 |                   |
  beforeShowDay.ts      |      100 |      100 |       80 |      100 |                   |
 tests                  |      100 |      100 |      100 |      100 |                   |
  beforeShowDay.test.ts |      100 |      100 |      100 |      100 |                   |
  calculator.test.ts    |      100 |      100 |      100 |      100 |                   |
------------------------|----------|----------|----------|----------|-------------------|
```

## 完整 Test Cases  

>  今天是 2019/3/30 號星期六 12:05  
>    √ 日曆上 2019/4/02 星期二 出貨 可以選  
>    √ 日曆上 2019/4/02 星期二 設定為國定假日, 出貨 不可以選  
>
>  今天是 2019/3/24 號星期日 12:05  
>    √ 日曆上 2019/3/25 星期一 出貨 不可以選  
>    √ 日曆上 2019/3/26 星期二 出貨 可以選  
>
>  今天是 2019/3/23 號星期六 12:05  
>    √ 日曆上 2019/3/25 星期一 出貨 不可以選  
>    √ 日曆上 2019/3/26 星期二 出貨 可以選  
>
>  今天是 2019/3/22 號星期五 23:05  
>    √ 日曆上 2019/3/25 星期一 出貨 不可以選  
>    √ 日曆上 2019/3/26 星期二 出貨 可以選  
>
>  今天是 2019/3/21 號星期四 01:30  
>    √ 日曆上 2019/3/21 星期四 出貨 不可以選  
>    √ 日曆上 2019/3/22 星期五 出貨 可以選  
>
>  今天是 2019/3/22 號星期五 23:05  
>    √ 日曆上 2019/3/25 星期一 出貨 不可以選  
>    √ 日曆上 2019/3/26 星期二 出貨 可以選  
>
>  今天是 2019/3/19 號星期二 23:05  
>    √ 日曆上 2019/3/21 星期四 出貨 可以選  
>
>  今天是 2019/3/22 號星期五 12:59  
>    √ 日曆上 2019/3/22 星期五 出貨 不能選,因為現在時間超過 12 點  
>    √ 日曆上 2019/3/25 星期一 出貨 不能選,因為現在時間超過 12 點  
>    √ 日曆上 2019/3/26 星期二 出貨 可以選  
>    √ 日曆上 2019/4/1 星期一 出貨 可以選  
>    √ 日曆上 2019/3/23 星期六 出貨 不能選,因為現在時間超過 12 點  
>
>  今天是 2019/3/21 號星期四 23:59  
>    √ 日曆上 2019/3/21 星期四 出貨 不能選,因為現在時間超過 12 點  
>
>  今天是 2019/3/18 號星期一12:00  
>    √ 日曆上 2019/3/18 星期一 出貨 不能選,因為現在是12點  
>    √ 日曆上 2019/3/19 星期二 出貨 不能選,因為現在是12點  
>    √ 日曆上 2019/3/20 星期三 出貨 可以選  
>    √ 日曆上 2019/3/21 星期四 出貨 可以選  
>
>  今天是 2019/3/18 號星期一10:00  
>    √ 日曆上 2019/3/24 星期日 出貨;不能選,因為週日都不能選  
>    √ 日曆上 2019/3/18 星期一 出貨 不可以選,因為當天不能選  
>    √ 日曆上 2019/3/19 星期二 出貨 可以選  
>    √ 日曆上 2019/3/20 星期三 出貨 可以選  

## 心得

能堅持「工序」是專業人士的表現，  
我相信這是一種實務上能保持品質與速度的作法。  
特別是越大規模越複雜的專案，  
避免掉進焦油坑的方法就是一開始就別踩下去。  

寫測試案例也是一種技能，在這次的 Case 中，  
E2E測試是相對困難的，受限於時間與日期，  
單元測試可以控制時間反而成了絕妙的工具。

我一開始寫的測試並不優良，涵蓋的情境不夠(註:這裡並非指程式碼的函蓋率)，  
但是與 PO 反覆確認之後，調試出的情境終於滿足了需求，  
看來我還要增進一下寫測試案例的能力。

## 參考
 - [TSUnitTestsSetup](http:// https://github.com/ChiragRupani/TSUnitTestsSetup)
 - [Writing unit tests in TypeScript](https://medium.com/@RupaniChirag/writing-unit-tests-in-typescript-d4719b8a0a40)

(fin)
