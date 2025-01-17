---
title: '[C#] String 轉 DateTime'
date: 2009-07-23 19:39:28
updated: 2009-07-23 19:39:28
categories:
- Backend
- C#
tags:
- C#
---
今天同事講到...
String在轉DateTime的時候...
要先轉double再轉...
感覺上蠻複雜的...

<!--more-->

其實只要簡單的兩行就可以了...
``` csharp
IFormatProvider culture = new System.Globalization.CultureInfo("zh-TW", true);
DateTime time = DateTime.ParseExact("20090723194530", "yyyyMMddHHmmss", culture);
```
中間有任何變化的話...
請自行搭配時間格式去設定...