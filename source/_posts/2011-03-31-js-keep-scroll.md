---
title: '[JS] 在Reload頁面時，保持ScrollBar的位置'
date: 2011-03-31 09:58:31
categories:
- Front-end
- JS
tags:
- js
---
最近的案子有一個需求
要在頁面Reload時保持ScrollBar的位置

<!--more-->

作法其實很簡單
就是在ScrollBar移動時將位置記住
每次Reload時去抓取存放的位置

我的作法是用一個暫存位置用的TextBox做儲存

``` js
$(document).ready(function(){
    var ttbScroll = $('#ttbScroll');
    var divMenu = document.getElementById('divMenu');
 
    if(ttbScroll.val() != '')
    {
        var split = ttbScroll.val().split(',');
        divMenu.scrollLeft = split[0];
        divMenu.scrollTop = split[1];
    }
 
    $('#divMenu').scroll(function(){
        ttbScroll.val(divMenu.scrollLeft + ',' + divMenu.scrollTop);
    });
});
```