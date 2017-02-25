---
title: 查詢正在執行的JOB及砍掉執行中的Session
date: 2013-02-23 20:38:39
categories:
- Back-end
- Oracle
tags:
- oracle
---
有時候Procedure/Package沒寫好時，會跑超級久
這時候會開始寫log或其他的動作來看調教效能

我們要先停下JOB才能夠繼續編輯，可以用以下步驟做

``` sql
--1.先確認一下JOB的資訊
select * from dba_jobs;

--2.確認正在執行的JOB資訊
select * from dba_jobs_running;

--3.停止JOB
exec dbms_job.broken('&JOB',true);

--4.當JOB還是繼續跑的時候，刪除還在執行的Session
select SID,SERIAL# from V$Session where SID='&SID';
alter system kill session '&SID, &SERIAL';

--5.刪除JOB（需要sysdba權限）
delete from dba_jobs where JOB='&JOB';
```

PS:&JOB、&SID、&SERIAL，表示select出來的欄位值