---
title: '[jQuery] live改用on寫法'
date: 2013-11-14 19:47:55
updated: 2013-11-14 19:47:55
categories:
- Frontend
- jQuery
tags:
- jQuery
---
jQuery在1.7版的時候就把live標記為deprecated，而建議改用on

<!--more-->

之前使用的時候都沒特別注意到差異
但最近使用到了live(這樣算是老人嗎XD)，但為了要跟上潮流
改用on，卻發生悲劇了，後來產生的沒有被綁上事件

``` js
//原本寫法
$('a.open').live('click', ToggleDetail);
//改用新寫法
$('a.open').on('click', ToggleDetail);
```

仔細查了一下jQuery官網，才發現原來搞錯寫法了on()
原來他們覺得原先的寫法，會讓人錯誤的以為事件的綁定方式，所以要正規化（我已經誤入歧途了orz）
自動綁定所有符合條件物件的事件，應該是從body或document開始去掃描
所以應該是要改成下面的寫法

``` js
//原本寫法
$('a.open').live('click', ToggleDetail);
//正確寫法
$(document).on('click','a.open', ToggleDetail);
```