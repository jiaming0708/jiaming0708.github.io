---
title: 第一個directive
date: 2014-11-25 19:32:03
categories:
- Front-end
- AngularJS
tags:
- AngularJS
---
同事需要一個按下enter轉成tab的功能，這在JS其實蠻好做的
但是在angular的$event屬性都是唯獨，導致這個功能沒有辦法直接實作

我改用另一個想法去實作這個功能
使用directive套用在element身上，並且找到下一個有宣告的element
為了能在enter後去呼叫API，所以再加上call back的功能

這是第一個實作出來且符合需求的directive <(￣︶￣)> 得意叉腰

{% gist acdc466842b6f7332455848145535b75 enterToTab.js  %}