---
title: '[Design Pattern] facade(外觀) 和 interpreter(直繹器) 以及 mediator(協調者)'
date: 2021-08-06 10:29:54
updated: 2021-08-06 10:29:54
categories:
- Design Pattern
tags:
- Design Pattern
---

繼上篇  {% post_link design-pattern-observer-decorator %}，我們接著繼續看其他的模式

> 迷之音:也隔太久了吧!!!

* Facade 外觀
* Mediator 協調者
* Interpreter 直譯器

<!-- more -->

# Facade (外觀)

## 目的

這個模式重點是封裝，只讓使用者看到外觀(表層)，這件事情其實大家都很常在做，尤其是當某個事件一定要執行固定邏輯時，那這樣我們就會想要把那些邏輯集中起來，讓外面呼叫端可以固定行為。

## 情境

那用一點現實的例子來看，用便當商店買東西來做說明，你可能會買咖啡/便當或是其他東西，不管哪一種的行為都是只有對應店員，剩下的煮咖啡/加熱，都會由店員來處理。

## 實作

{% gist 43901411a5164db71ace1d75d2d10cef facade.cs %}

> ❯  dotnet run
> ---買餅乾---
> 結帳囉
> ---買咖啡---
> 結帳囉
> 咖啡好了
> ---買便當---
> 結帳囉
> 加熱完成

# Mediator (協調者)

## 目的

將原本互相交錯的溝通，向上抽一層，統一由第三方來轉接收並且轉交。

## 情境

以現實的世界來看，其實工作中就很多這樣的狀況，工程師、設計師、行銷，老闆除了要跟這些人說明他要的是什麼，同時也要接收他們的狀況回報，並且這些人彼此間也需要互相更新狀態。
對於老闆來說，不需要一直收到那麼多的訊息，這時候我們會獨立一個角色叫做 **PM** ，由這個人來負責所有的資訊，彼此間也不用丟出各種訊息給不必要的人，統一由新的角色來做處理，對於老闆來說，也只要叫PM回報狀況就好。

## 實作

{% gist 43901411a5164db71ace1d75d2d10cef mediator.cs %}

> ❯  dotnet run
> designer receive login completed from Engineer
> marketing receive login completed from Engineer
> engineer receive home page mockup ready from Designer
> marketing receive home page mockup ready from Designer

# Interpreter (直譯器)

## 目的

偶爾在系統中，會需要一些簡單的文字解析，這時候可以使用這個設計模式來輔助我們，讓後面要擴充的時候更為容易。
但效能不是第一考量，因此有效能的需求就應該要重新思考是否適用。

## 情境

有一組字串，裡面的資料可能要轉換成中文或是英文

## 實作

{% gist 43901411a5164db71ace1d75d2d10cef interpreter.cs %}

> ❯  dotnet run
> 二零二一 to 2021
> 2021 to 二零二一
