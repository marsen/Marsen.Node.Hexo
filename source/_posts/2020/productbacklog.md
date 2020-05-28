---
title: "[A社筆記] Product Backlog "
date: 2020/05/28 15:06:02
tag:
    - Aglie
---

## 前言

> Although implementing only parts of Scrum is possible, the result is not Scrum.
> >
> 雖然實施部分的 Scrum 是可能的，但結果並不是 Scrum 《Scrum Guide》

之所以在文章一開頭就引述這段文字，是因為世界上太多 `Scrum But` 了;  
而我下文所敘的則是 `Not Scrum`。  
未來有機會再深入討論兩者的差異，對我來說主要是心態上的差異。

我們透明、檢核、調試，我們擁抱改變，我們有所堅持，我們有理想但我們也務實。  
好啦，說不定只有你自已。  

在 A 社的導入的過程中，我開場白通常會說「這不是 Scurm」然後開始拿 Scurm 的東西說嘴。  
至少對我來說，這比那些號稱「我們跑 Scrum」「But #^%!~…」要好得多，  
與其讓你難以分辨斷句在哪，或是誤會原來 Scrum 這麼糞，不如讓我明確的告訴你「這不是 Scurm」。  

## 背景

蠻小巧的團隊，小到我覺得也許不用任何「敏捷」  
分為兩個組成 QA 部門與 RD 部門，遠端工作的 RD 主管兼職 PM  
QA 團隊的主管除了目前這個團隊，也要兼者作其它團隊的測試，  
此外還有一個大主管，在國外有時差，每周固定會與成員們開 3 次的會。  
RD 團隊成為會有一個 Daily Sync 的會議，每天約 5~10 分鐘。
整個團隊有使用 Azure DevOps 的看板，但是 QA 只會用來開 Bug。
但不知道什麼時候 Bug 才會被修復。

## 問題

### Azure DevOps BackLogs 太像文件

Azure DevOps 的 BackLogs 放著許多 Feature ，  
依據不同的功能，再將每個 Sprint 的 User Story 放進去
這些 Feature 永遠不會被作完，隨著時間過去，  
儘管完成了許多 User Story ，但是也會有新的 User Story 被加進去。
而作為文件，他的巢狀結構又不足以面對複雜的需求內容，  
分散式存儲對於維護與修改上也是相同不便。

### Bug 隨時蹦出

QA 的工作就是不斷的測試，一有發現問題就開立 Bug，  
RD 有空就會去領來作，有時候也會有 RD 主管確認過後再分配給 RD  
這導致不同面向的問題，RD 自領的情況可能會無序的作，
導致真正有價值的 Bug 無法第一時間被修正。
而如果每件事情都要 RD 主管確認後再進行動作，  
RD 主管恐怕會變成瓶頸所在。

## BackLogs 簡介

在 Azure DevOps 上會有 BackLogs 與 Sprints > Backlog。
恰巧與 Scurm 的 Product BackLogs 與 Sprints Backlog 可以一一對應。
如果以 XP 來說，可以投射到發佈計劃會議與迭代計劃會議中討論的「用戶故事清單」。
如果以 GTD 的方法論來說 BackLogs 可以當作收集一切事務的 Inbox，
而 Sprints > Backlog 可以視作專案裡面的工作項目。

總的來說，這些方法論的名詞故有所不同，但本質上卻是非常接近的事務。
這裡介紹一下遠光燈的觀念…

(fin)
