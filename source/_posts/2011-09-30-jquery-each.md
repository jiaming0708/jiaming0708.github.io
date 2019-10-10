---
title: '[jQuery] each迴圈的continue/break'
date: 2011-09-30 11:54:16
categories:
- Front-end
- jQuery
tags:
- jQuery
---
在jQuery中使用each時，要進行continue/break只要這樣做就行了
<!--more-->

``` js
$('tag').each(p=>{
    //continue
    return true;
    //break
    //return false;
})
```