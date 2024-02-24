---
title: " [學習筆記] Vim 學習資源"
date: 2024/02/22 14:01:17
---

## 介紹

Vim 是一款強大高效的文本編輯器，強調效率和快速操作。  
學習 Vim 能顯著提升編輯速度和效率，無需滑鼠，節省操作時間。  
這裡記錄著我的學習資源，提供給未來的我複習使用。

## TLDR Tips

1. `gg`：移動到檔案的第一行
2. `G`：移動到檔案的最後一行
3. `gg=G`：重新縮排整個檔案
4. `gv`：重新選取上一次的視覺選取
5. `` <`：跳到上一次視覺選取的開始
6. `` >`：跳到上一次視覺選取的結尾
7. `^`：移動到行的第一個非空白字符
8. `g_`：移動到行的最後一個非空白字符
9. `g_lD`：刪除行上的所有尾部空白
10. `ea`：在當前單字的末尾插入
11. `gf`：跳到游標下的文件名
12. `xp`：向前交換字符
13. `Xp`：向後交換字符
14. `yyp`：複製當前行
15. `yapP`：複製當前段落
16. `dat`：刪除包括標籤在內的 HTML 標籤
17. `dit`：刪除 HTML 標籤內的內容，但不包括標籤本身
18. `w`：向右移動一個單字
19. `b`：向左移動一個單字
20. `dd`：刪除當前行
21. `zc`：關閉當前摺疊
22. `zo`：打開當前摺疊
23. `za`：切換當前摺疊
24. `zi`：完全切換摺疊
25. `<<`：向左移動當前行的縮排
26. `>>`：向右移動當前行的縮排
27. `z=`：顯示拼寫更正
28. `zg`：添加到拼寫字典
29. `zw`：從拼寫字典中刪除
30. `~`：切換當前字符的大小寫
31. `gUw`：將大小寫轉換到單字的末尾（u 用於小寫，~ 用於切換）
32. `gUiw`：將整個單字轉換為大寫（u 用於小寫，~ 用於切換）
33. `gUU`：將整行轉換為大寫
34. `gu$`：將直到行尾的文本轉換為小寫
35. `da"`：刪除下一個雙引號括起來的字符串
36. `+`：移動到下一行的第一個非空白字符
37. `S`：刪除當前行並進入插入模式
38. `I`：在行的開頭插入
39. `ci"`：更改下一個雙引號括起來的字符串內容
40. `ca{`：更改大括號內的內容（也可以試試 [, ( 等）
41. `vaw`：視覺選取單字
42. `dap`：刪除整個段落
43. `r`：替換字符
44. ``[`：跳轉到上次複製的文本的開始
45. ``]`：跳轉到上次複製的文本的結尾
46. `g;`：跳轉到上次更改的位置
47. `g,`：向前跳轉到更改列表
48. `&`：在當前行上重複上次的替換
49. `g&`：在所有行上重複上次的替換
50. `ZZ`：儲存當前檔案並關閉

## 學習資源

內建學習工具

```terminal
> vimtutor
```

- [50 Useful Vim Commands](https://vimtricks.com/p/50-useful-vim-commands/)
- [影片:即將失傳的古老技藝 Vim --- 高見龍](https://www.youtube.com/playlist?list=PLBd8JGCAcUAH56L2CYF7SmWJYKwHQYUDI)
- [網站:Vim 的操作小技巧](https://kaochenlong.com/2011/12/28/vim-tips/)
- [Vim Golf](https://www.vimgolf.com/)

(fin)