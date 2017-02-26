---
title: 寫入的簡體資料變成亂碼
date: 2012-02-17 15:44:05
categories:
- Back-end
- Oracle
tags:
- oracle
---
最近用自己的DB去測試程式時，發現簡體的資料存進去都變成空白字元
在網路上查詢了一下，是因為DB的一個設定導致
可使用底下方法修改，資料儲存就正常了

使用Regedit找到
> `HKEY_LOCAL_MACHINE\SOFTWARE\ORACLE\KEY_ORADB11G_HOME1\NLS_LANG`
原始值為TRADITIONAL CHINESE_TAIWAN.ZHT16MSWIN950
修改值為AMERICAN_AMERICA.AL32UTF8