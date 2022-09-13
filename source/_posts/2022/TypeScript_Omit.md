---
title: "[學習筆記] TypeScript Omit 的用法"
date: 2022/09/12 10:34:53
tag:
  - TypeScript
---

## 簡介

本文簡單介紹一下 TypeScript Omit 的用法與範例，
更多資料可以參考[官方文件](https://www.typescriptlang.org/docs/handbook/utility-types.html)

## 說明

Omit 字義為忽略、省略，在 TypeScript 中是一種 Utility Type,  
使用方法如下:

```typescript
type NewType = Omit<OldType, "name" | "age">;
```

第一個參數是傳入的 Type, 第二個參數是要忽略的欄位，
並會回傳一個新的 Type, 這是 TypeScript Utility Types 的標準用法。

具體一個例子

```typescript
interface LocationX {
  address: string;
  longitude: number;
  latitude: number;
}

interface SpecificLocation extends Omit<LocationX, "longitude" | "latitude"> {
  coordinate: {
    longitude: number;
    latitude: number;
  };
}

type AnotherLocation = Omit<SpecificLocation, "coordinate"> & {
  x: number;
  y: number;
};

const example1: SpecificLocation = {
  address: "example first on some where",
  coordinate: {
    latitude: 100,
    longitude: 100,
  },
};

const example2: AnotherLocation = {
  address: "example 2nd on some where",
};

console.log(example1);
console.log(example2);
```

說明:

1. `interface` 可以用 `extends` 加上新欄位，借此實現 override 欄位的功能
2. `type` 使用 `&` 作擴展，一樣的方式先省略(Omit)再擴展欄位
3. 相反的 Utility Type 有 [`Pick`](https://www.typescriptlang.org/docs/handbook/utility-types.html#picktype-keys)

[完整範例](https://codesandbox.io/s/typescript-omit-sample-etoc9s)

這些 Utility Types 蠻有趣的，有時間應該將它們補完。

## 20220913 補充

進一步探討 `Omit`

1. `Omit` 適用於 `type` 與 `interface`
2. 與 `Omit` 相反的就是 `Pick`
3. `Omit` 與 `Exclude` 的差別

   - `Omit` 移除指定 Type 的欄位
   - `Exclude` 移除 union literal Type 當中的成員
   - `Exclude` 可以適用於 `enum`

舉例說明:

下面兩個例子分別使用了 `union literal` 與 `enum`,  
在這種情況下如需要忽略部份的成員, 請使用 `Exclude`,  
你要用 `Omit` 也不會報錯, 但是無法達到編譯時期警告的效果

```typescript
type TrafficLight = "red" | "yellow" | "green";

type RedLight = Exclude<TrafficLight, "yellow" | "green">;
type GreenLight = Omit<TrafficLight, "red">;

const sighs: {
  Danger: RedLight;
  Warning: TrafficLight;
  Safe: GreenLight;
} = {
  Danger: "red", // if you use other words would get error
  Warning: "yellow",
  Safe: "there is no constraint any string",
};

console.log(sighs);
```

[範例](https://codesandbox.io/s/typescrip-utiliy-types-exclude-union-literal-g2tp73?file=/src/index.ts)

```typescript
enum TrafficLight {
  red,
  yellow,
  green,
}

type RedLight = Exclude<TrafficLight, TrafficLight.yellow | TrafficLight.green>;
type GreenLight = Omit<TrafficLight, TrafficLight.red>;

const sighs: {
  Danger: RedLight;
  Warning: TrafficLight;
  Safe: GreenLight;
} = {
  Danger: TrafficLight.red, // if you use yellow or green would get error
  Warning: TrafficLight.yellow,
  Safe: TrafficLight.red,
};

console.log(sighs);
```

[範例](https://codesandbox.io/s/typescrip-utiliy-types-exclude-56h2ni?file=/src/index.ts)

## 參考

- <https://bobbyhadz.com/blog/typescript-override-interface-property>
- <https://timmousk.com/blog/typescript-omit/>
- <https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys>
- <https://ithelp.ithome.com.tw/articles/10269471>

(fin)
