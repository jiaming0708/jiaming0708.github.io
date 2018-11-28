---
title: Design Pattern-Observer(觀察者)、Decoractor(裝飾器)
date: 2017-08-29 20:17:54
categories:
- Design Pattern
tags:
- Design Pattern
---

今天要來談談不一樣的東西，關於Design Pattern的兩三事，在程式開發中應用了這些模式在撰寫上可以更輕鬆一點，那我們先來看看其中的兩個

* Observer 觀察者
* Decoractor 裝飾器

<!--more-->

# Observer Pattern

當我們今天要去確認一個目標有沒有新的資料，我們必須要不斷的詢問，不只詢問的人很煩，被問的人也要花費力氣去回應(儘管這可能很輕鬆)，但很多人跟他詢問的時候就會佔用太多時間

![request](./request.png)

那這時候其實我們可以反過來思考，請求者跟目標說，你有新的資料就通知我，這種情況下雙方都會比較輕鬆，那這種模式其實就是Observer Pattern

![observer](./observer.png)

那我們最常聽到的範例就是跟報社訂報紙，訂戶跟報社訂閱報紙，只要報社一出版就會送到訂戶手上，也就是說訂戶是被動而報社是主動

根據上述的內容，我們來試著實作看看

{% gist eeb7fb552d14a06690665ba27a013154 Producer.ts %}

{% gist eeb7fb552d14a06690665ba27a013154 main.ts %}

> A new course!!from listener1
> A new course!!from listener2

# Decoractor Pattern

今天買了一台摩托車，很帥的想要去把妹，但是缺少了點個性化，所以我們可以加上一些商家已經做好的套件裝上去，例如換沖天排氣管、翹牌器
這個精神可以稱作修飾(decoractor)

那接著來實作看看吧

{% gist 8f9d48ec2263994ac325c1d407260fab decoractor.ts %}

> f(): evaluated
> g(): evaluated
> g(): called
> f(): called

