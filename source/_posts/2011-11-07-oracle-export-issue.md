---
title: '[oracle] 使用者可以查詢卻無法匯出'
date: 2011-11-07 18:46:30
updated: 2011-11-07 18:46:30
categories:
- Backend
- Oracle
tags:
- Oracle
---
最近遇到了一個情況

<!--more-->

客戶為了DB的安全性，將密碼做了修改
改了以後執行Export和Import動作時，出現了
> ORA-12154:TNS:無法解析指定的連接標示符

但是登入使用者時卻又可以登入，也可以查詢資料

改用測試帳號執行匯出，發現正常沒有問題
將測試帳號的密碼改為和正式的密碼相同，也不能執行匯出
發現是密碼中帶有@的特殊字元，才無法執行
在oracle中，@在DBLink中會使用到