---
title: '[nginx] 運行於 windows 底下設定 log 的備份機制'
date: 2023-12-15 15:58:03
updated: 2023-12-15 15:58:03
categories:
- nginx
tags:
- nginx
thumbnail:
---

先前有介紹過 {% post_link logrotate-nginx %}，這是基於 linux 的環境，假如你的 nginx 是運行在 windows 底下的話，沒有套件可以做這件事情，今天就來分享一下可以怎麼做。

<!-- more -->

在 windows 中，雖然沒有套件可以使用，還是可以用最老牌的功能`工作排程器`搭配腳本來達成目的。

腳本的概念是這樣，參照 linux 的 logrotate 作法，檔案超過一定大小或是換日就做備份。以這樣的邏輯工作排程要設定每分鐘或每天的檢查，就都不是問題了。

{% gist 3e8cb7252f350ccdaeb595984595acd9 nginx-logrotate-windows.ps1 %}

> 這邊要注意的是執行的時候必須要 `最高權限(Administrator)`，原因是最後要將 nginx 的服務重啟。