---
title: 連結MySql
date: 2012-01-13 10:12:16
categories:
- Back-end
- C#
tags:
- C#
---
最近case是要和MySql連結，之前都沒有用過，特別記一下

<!--more-->

MySql和Oracle類似，都必須先安裝Client才能夠連線
[MySql下載](http://dev.mysql.com/downloads/)

特別要注意一下
如果server版本比較舊，請考慮不要抓最新版本

我遇到的server版本是4.0.18，client裝的版本是5.1，連線後出現下面的錯誤訊息
> DBS-00000 : ERROR [08001] [MySQL][ODBC 5.1 Driver]Driver does not support server versions under 4.1.1

後來client直接改裝3.X版本，連線正常

MySql的連線可採用OleDb的元件，或是採用另一個MySql元件去做連線