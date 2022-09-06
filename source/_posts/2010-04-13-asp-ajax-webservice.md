---
title: '[ASP.NET] ajax使用WebService'
date: 2010-04-13 20:25:45
categories:
- Backend
- ASP.NET
tags:
- ASP.NET
---
最近用WebService來做一些功能
很奇怪的再local測試都沒問題
但一放到server上就有問題

<!--more-->

錯誤訊息沒存起來
不過大概是說傳遞格式出錯

網路找一找才發現
原來要再`web.config`中加上這段

``` xml
<system.web>
<webServices>
<protocols>
<add name="HttpPost"/>
<add name="HttpGet"/>
</protocols>
</webServices>
```