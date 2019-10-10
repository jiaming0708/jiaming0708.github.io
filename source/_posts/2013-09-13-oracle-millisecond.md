---
title: '[oracle] 取得時間到毫秒'
date: 2013-09-13 21:14:25
categories:
- Back-end
- Oracle
tags:
- oracle
---
記錄時間的時候，通常會用to_char(sysdate, 'yyyy/MM/dd HH24:mi:ss')
但這樣只能顯示到秒，無法在顯示到更小的單位

<!--more-->

當想要看到毫秒的話，就不能使用sysdate
要改用systimestamp，to_char(systimestamp, 'yyyy/MM/dd hh24:mi:ssxff')

就可以到這樣的資料 2013/02/23 21:20:32.299000
就可以更精準的做想要看的資料了