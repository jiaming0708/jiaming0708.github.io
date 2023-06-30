---
title: '[Debug] 前端如何偵錯CORS'
date: 2023-06-30 16:21:56
updated: 2023-06-30 16:21:56
categories:
- Frontend
tags:
- Frontend
- Debug
thumbnail:
---

前端的社群朋友遇到一個神奇的問題，在 axios 就會有 cors 錯誤，改使用 fetch 就不會，產生一些討論以後，覺得如何偵錯 cors 蠻值得紀錄的。

先說結論，只要遇到 CORS 一律都是後端的問題。
> 本篇結束(大誤

<!-- more -->

## 介紹

### 什麼是CORS

全名叫做 **Cross-Origin Resource Sharing** 簡稱 CORS，只有在瀏覽器才會有的行為，其它後端打後端或是 APP 打後端都不會有這問題的發生。
會發生的主要狀況是只要你請求資源不是同一個 `protocol`/`domain`/`port`，瀏覽器就會先發一個請求去伺服器確認是否允許。

| web server | api server | CORS | 說明 |
| ---- | ---- | ---- | ---- |
| http://localhost:3000 | http://localhost:3001 | V | port 不同 |
| http://localhost:3000 | https://localhost:3000 | V | protocol 不同 |
| http://localhost:3000 | http://api.test.com | V | domain/port 不同 |
| http://localhost:3000 | http://localhost:3000 | X |  |

在這種情況下，從 `DevTool` 的 `network` 可以看到有兩個相同的請求出去但方法不同，第一個就是 `options`，第二個才會是你真正的請求。

只要一律到 CORS 的錯誤，其實就代表目標伺服器不允許，如果是自己家的就可以叫後端工程師調整；但如果是外面那種 open API，就必須要先打到自己的 server 再由 server 去呼叫，避開這個問題。

### 後端常見設定

以下三種是後端比較常見的設定:

- Access-Control-Allow-Origin，請求的來源位置
- Access-Control-Allow-Methods，請求的來源方法，put/get/delete/patch...等等的
- Access-Control-Allow-Headers，額外的 header 內容

## 偵錯技巧

身為前端的你，到底該如何去看是什麼鬼東西被卡住呢，根據朋友的案例，可以有兩種做法。

### 追程式

把兩種寫法的程式都看清楚，並且比對有哪邊行為不一樣，這個通常會比較考驗眼力和耐心。

### 使用開發者工具

打開瀏覽器的 `DevTool`，並且切到 `Console`，就會看到紅色的字告訴你請求失敗。

![console-error](console-error.jpg)

像是上面這樣的畫面，裡面就包含著答案 **Request header field `authorization` is not allowed by `Access-Control-Allow-Headers` in preflight response.** 。

- `header filed` 後面所接的就是不被允許的欄位
- `Access-Control-Allow-Header` 也就是前面所說額外 header 內容

> 基本上 authorization 不被允許，這個後端要抓來打屁股

## Reference

- [跨來源資源共用（CORS） - HTTP | MDN (mozilla.org)](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/CORS)
