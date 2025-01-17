---
title: '[Cordova] hooks功能-檢查、壓縮、合併'
date: 2016-12-23 10:51:17
updated: 2016-12-23 10:51:17
categories:
- APP
- Cordova
tags:
- Cordova
---

最近才得知cordova可以在編譯輸出前做一些事情
這個地方叫做hooks，可以到[官網](https://cordova.apache.org/docs/en/latest/guide/appdev/hooks/)查詢

<!--more-->

預計要來做幾件事情
1.JSHint
2.Concat files
3.Uglify

## JSHint

檢查JS語法，這個會放在before_prepare，要讓錯誤扼殺在成長之前XD
所以我們要建立一個檔案，`010_jshint.js`

hooks的運作機制，會根據js前面的號碼作為排序，來依序執行檔案
有多個動作時，記得要確認一下要執行的順序為何

{% gist ad926b08234864967c36a72950eb86d1 010_jshint.js %}

## Concat files

撰寫時，我們會將js、css根據模組或功能等需求而分開來
等到我們要執行時，為了效能的優化，就會將檔案進行合併

這個動作會放在after_prepare下，這邊也要建立一個檔案`012_concat_files_by_index.js`
這功能主要是將index.html中所宣告引用的js、css進行合併，所以index.html還是要記得先加上

{% gist ad926b08234864967c36a72950eb86d1 012_concat_files_by_index.js %}

做完這段以後可以發現到，index.html中已經被合併的檔案tag還是存在，那接著我們就要對於這些被合併的tag換成最後輸出的檔案
那這邊需要先對index.html中加上一些東西，才能讓接下來的功能順利進行，可以參考[node-useref](https://github.com/digisfera/useref)

```html
<!-- build:css css/app.min.css -->
<link href="css/ionic.app.css" rel="stylesheet" />
<link href="css/style.css" rel="stylesheet" />
<!-- endbuild -->
<!-- build:js js/app.min.js -->
<script src="js/app.js"></script>
<!-- endbuild -->
```
一樣在after_prepare中，加上`015_prepare_index.js`

{% gist ad926b08234864967c36a72950eb86d1 015_prepare_index.js %}

## Uglify

這是裡面最簡單的好做到的功能XD，只要下個指令就可以
會自動在after_prepare下增加一個`uglify.js`檔案，因為前面沒有順序，所以會是最後執行
詳細內容一樣可以到[官網](https://github.com/rossmartin/cordova-uglify)中查詢

``` bash
npm install cordova-uglify --save
```

做完以上的動作，我們就可以下個指令確認一下

``` bash
cordova prepare [platform] --release
```

沒有看到任何錯誤就是成功啦
可以打完收工囉

ㄟㄟㄟ～等等！
mac用戶明明會遇到另一個錯誤

好吧～我只好再回來繼續
因為mac用戶會有權限的問題，會只看到一個鬼一般的訊息
>Error: EACCES

那這個問題不大，只要下個指令給個權限就好，針對我們在after_prepare所新增的兩個檔案012、015

``` bash
chmod +x hooks/after_prepare/[filename]
```
好啦，這次真的要打完收工了