---
title: 獨立交易，不影響外層的交易行為
date: 2013-02-23 22:57:33
categories:
- Back-end
- Oracle
tags:
- oracle
---
當要調教效能時，會採用寫log的方式
但是寫log是一定要commit，但commit下去一定會影響到外面在做的事情

這時候怎麼辦呢，還好同事有經驗
這種功能在Oracle中，中文叫做自治事務，英文是Autonomous Transactions

要怎麼應用勒，其實很簡單
在要獨立交易的Procedure中多加一行 PRAGMA AUTONOMOUS_TRANSACTION;
還是看一下實際範例吧

``` sql
PROCEDURE WriteLog (text IN VARCHAR2)
IS
 
PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN

INSERT INTO LOG VALUES (text, SYSDATE);
COMMIT;

END;
```