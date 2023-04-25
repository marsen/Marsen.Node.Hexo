---
title: " [實作筆記] Dialog Button 實作 React 父子元件交互作用"
date: 2022/08/22 15:46:00
tag:
  - 實作筆記
  - React
---

## 前情提要

最近我設計了一個 Dialog Button ，過程十分有趣，稍微記錄一下

### 需求

首先 Dialog Button 的設計上是一個按鈕，被點擊後會開啟一個 Dialog，  
它可以是表單、一個頁面、另一個組件或是一群組件的集合。  
簡單的說 Dialog 可以是任何的 Component。

用圖說明的話，如下

![Dialog Button](https://i.imgur.com/aC3tUjs.png)

藍色表示父層組件，我們會把 Dialog Button 放在這裡
綠色就是 Dialog Button 本身，它會有兩種狀態，Dialog 的隱藏或顯示
紅色則是任意被放入 Dialog Button 的 Component。  
也就是說我們會用 attribute 傳遞 Component 給 Dialog Button

注意黃線的部份，我們會幾種互動的行為

1. Component 自身的行為，比如說計數器 Component 的計數功能
2. Component 與 Dialog Button 的互動，比如說關閉 Dialog Button
3. Component 與 Dialog Button 的 Parents Component 互動，比如說重新 Fetch 一次資料

具體的 Use Case 如下，  
我在一個表格(Parents Component)中找到一筆資料，  
點下編輯(Dialog Button)時會跳出一個編輯器(Component)，  
編輯器在我錯誤輸入時會提示<sup>1</sup> 警告訊息，  
在修改完畢儲存成功時，會關閉編輯器<sup>2</sup>並且同時刷新表格資料<sup>3</sup>
參考下圖  
![count、close and fetch ](https://i.imgur.com/LESekFn.png)

## 實作

首先先作一個陽春的計數器，這個是未來要放入我們 Dialog 之中的 Component,  
這裡對有 React 經驗的人應該不難理解，我們透過 useState 的方法來與計數器互動

```tsx
export default function Counter() {
  const handleClick = () => {
    setCount(count + 1);
  };
  const [count, setCount] = useState(0);
  return (
    <>
      Count :{count}
      <Stack spacing={2} direction="row">
        <Button onClick={handleClick} variant="contained">
          Add
        </Button>
      </Stack>
    </>
  );
}
```

接下來我們來實作 Dialog Button，大部份的實作細節我們先跳過
我們來看看如何使用傳入的 Component 並將其與 Dialog 內部的函數銜接起來

```tsx
<DialogContent>
  {cloneElement(props.content, { closeDialog: closeDialog })}
</DialogContent>
```

這個神奇方法就是 [cloneElement](https://zh-hant.reactjs.org/docs/react-api.html#cloneelement)，
從官方文件可知，我們可以透過 config 參數提供 prop、key 與 ref 給 clone 出來的元素，

```tsx
React.cloneElement(element, [config], [...children]);
```

所以我們將 `closeDialog` 方法作為一個 props 傳遞給 dialog 內開啟的子組件，比如說 : Counter

完整程式如下:

```tsx
export function DialogButton(props: DialogButtonProps) {
  const [open, setOpen] = useState(false);
  const openDialog = () => {
    setOpen(true);
  };

  const closeDialog = () => {
    setOpen(false);
  };

  return (
    <>
      <Button {...props} onClick={openDialog}>
        {props.children}
      </Button>
      <Dialog
        open={open}
        onClose={closeDialog}
        maxWidth={props.max_width}
        fullWidth
      >
        <DialogTitle>
          {props.title}
          <IconButton
            aria-label="close"
            onClick={closeDialog}
            sx={{
              position: "absolute",
              right: 6,
              top: 6,
            }}
          >
            <CloseIcon />
          </IconButton>
        </DialogTitle>
        <DialogContent>
          {cloneElement(props.content, { closeDialog: closeDialog })}
        </DialogContent>
      </Dialog>
    </>
  );
}
```

最後我們父層元素，也可以將自身的方法傳遞給 content Component  
以達到 Component 與父層元素互動的功能，不需要再經過 Dialog Button 了

![parent interact with content](https://i.imgur.com/ePMLhjm.png)

## 參考

- [完整範例](https://codesandbox.io/s/great-worker-d2f2kc?file=/src/DialogButton.tsx)
- [cloneElement](https://zh-hant.reactjs.org/docs/react-api.html#cloneelement)

(fin)
