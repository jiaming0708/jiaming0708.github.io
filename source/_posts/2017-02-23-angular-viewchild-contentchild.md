---
title: '[Angular] ViewChild及ContentChild介紹'
date: 2017-02-23 22:10:58
updated: 2017-02-23 22:10:58
categories:
- Frontend
- Angular
tags:
- Angular
---

昨天參加[線上angular讀書會](https://www.facebook.com/groups/1345890212093830/)(打個廣告)，分享的內容是ngContent
之前自己在寫的時候，套用bootstrap的功能時就有應用到
但那時候一直看不懂是什麼意思，在昨天就整個豁然開朗

<!--more-->

那到底什麼是ViewChild、ContentChild，底下來看看範例吧

## ViewChild
{% gist af72b77db5c3c9e4ac66721e8c4ab46b app.component.ts %}

app-tab這個element就是ViewChild，也就是子Component的概念
這時候就會有人問，這樣做有什麼好處

可以直接引用子Component所有public事件、屬性
也就是說，可以不用辛苦的用Input、Output來進行傳遞(好壞先不討論囉!)

## ngContent
{% gist af72b77db5c3c9e4ac66721e8c4ab46b tab.component.ts %}

什麼是ngContent，可以說是叫做範本引入或是內容置換
也就是把那個位置的內容變成由外面傳進來，app.component.ts中app-tab的內容就是自定義

那基本上ng-content就是直接把外面傳的東西換掉放進去
進階一點的應用就是有多個ngContent，需要指定每個所放的位置
在ng-content中加上select並指定就可以取得父component所傳入的內容
這邊有三個類型
* .class
* tagName
* 沒有指定

## ContentChild
ContentChild有兩種取得的方式，在同一個parent中，是不能有重複的#id
* #id
* className

記得要取得ContentChild必須要在AfterContentInit這個事件之後才能拿到物件
你才能夠爽爽的使用，不然會鼠翹翹喔!

## Child vs Children
差異非常的小，就是一個單數一個複數，收工@@

## 參考資料
[線上angular讀書會 S01E03](https://youtu.be/TMvOew5h85Q)
[ViewChildren and ContentChildren in Angular 2](http://blog.mgechev.com/2016/01/23/angular2-viewchildren-contentchildren-difference-viewproviders/)