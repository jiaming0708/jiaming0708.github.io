---
title: '[C#] parse'
date: 2008-12-29 16:39:55
categories:
- Backend
- C#
tags:
- C#
---
## 顏色
將顏色代碼"#2952A3"要轉成Color型別
Color型別只有三種parse
FromArgb、FromKnownColor、FromName
後面兩種都是Color型別中已存在命名的顏色去做parse

<!--more-->

所以只能用第一種，但是他又只支援int32
那這種代碼要怎麼轉換
其實很明顯的，這種代碼是16進制
只要將16先轉成10就可以了
``` csharp
int.Parse("#2952A3".Replace("#", ""), System.Globalization.NumberStyles.AllowHexSpecifier)
```

## 日期
將字串"20081229T120000"轉成DateTime型別
DateTime的型別有兩種的parse
Parse、ParseExact
Parse適用於像這種較為明確的字串格式"2/16/1992 12:15:12"
而所得到的字串是不明確時，要採用ParseExact
``` csharp
IFormatProvider culture = new System.Globalization.CultureInfo("zh-CHT", true);
DateTime.ParseExact("20081229T120000".Replace("T", " "), "yyyyMMdd HHmmss", culture);
```