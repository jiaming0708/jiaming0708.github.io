---
title: '[oracle] 11G沒有資料的Schema無法匯出'
date: 2011-08-05 08:23:44
categories:
- Back-end
- Oracle
tags:
- oracle
---
Oracle在11G開始，匯出整個使用者時，沒有資料的Schema是不會匯出的
還好這是可以設定的，我們可以先用下面的語法檢查

<!--more-->

``` sql
show parameter deferred_segment_creation
```

結果為true，就不會匯出空的Schema
只要用下面的語法就可以讓空的Schema也匯出了

``` sql
alter system set deferred_segment_creation=false;
```