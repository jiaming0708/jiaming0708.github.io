---
title: Gitea 資料庫遷移
date: 2023-02-20 11:06:21
updated: 2023-02-20 11:06:21
categories:
- Infra
tags:
- Infra
- Gitea
thumbnail:
---

公司的 Gitea 原本資料庫是用 SQLServer，那台 Server 本身也比較舊，在有些操作下也會覺得不順，剛好有機會可以停機可以轉成 Postgres，這邊來記錄一下我的操作步驟。

<!-- more -->

## 備份指令介紹

我們可以透過指令的方式針對 Gitea 來做[備份和還原](https://docs.gitea.io/en-us/backup-and-restore/)，備份功能本身能夠支援輸出成不同資料庫，詳細的指令的話在文件上就沒看到，我是透過 [Migrate Gitea DB from MariaDB to PostgreSQL - Support - Gitea](https://discourse.gitea.io/t/migrate-gitea-db-from-mariadb-to-postgresql/2072/2) 這篇的討論和 [Source code](https://github.com/go-gitea/gitea/blob/main/cmd/dump.go) 來找到需要的指令。

對於 Gitea 來說，備份的內容包含 sql file、log、repository，需要做的只有資料庫的部分，省略其他的輸出可以讓備份的速度加快，最終的指令如下:

```shell
/app/gitea/gitea dump -d postgres -R -L -c /data/gitea/conf/app.ini
```

- database(d)，可以指定輸出的資料庫類型 `PostgreSQL`, `MySQL `, `SQLite`, `MSSQL`
- skip-repository(R)，略過 repository 備份
- skip-log(L)，略過 log 備份
- config(c)，指定 app.ini 檔案的路徑

## 執行備份

接著就要實際的下指令囉，在公司的環境中，是用 Docker 來做安裝，因此要先連到 Gitea 的 Container 中。

```shell
docker exec -it gitea bash
```

執行前面所介紹的指令，這邊會收到一個 permission 的錯誤，主要是不能使用 root 來做 dump 的指令。

```
2023/02/20 11:43:31 ...s/setting/setting.go:1059:loadFromConf() [F] Gitea is not supposed to be run as root. Sorry. If you need to use privileged TCP ports please instead use setcap and the `cap_net_bind_service` permission
```

這邊只要切換使用者為 `git` ，再次執行前面的指令。

```shell
su git
```

就可以看到開始備份的訊息，完成的時候會看到一個 `gitea-dump-*.zip` 的檔案名稱在你執行的目錄底下。

```shell
2023/02/20 03:55:26 ...dules/setting/log.go:288:newLogService() [I] Gitea v1.18.3 built with GNU Make 4.3, go1.19.5 : bindata, timetzdata, sqlite, sqlite_unlock_notify

...

2023/02/20 03:55:49 cmd/dump.go:405:runDump() [I] Finish dumping in file gitea-dump-1676865326.zip
```

那整合一下以上的作法，可以這樣下指令，一次匯出。

```shell
docker exec -it -u git -w /data $(docker ps -qf 'name=^gitea$') bash -c '/app/gitea/gitea dump -d postgres -R -L -c /data/gitea/conf/app.ini'
```

> -w 為工作目錄，因為 data 的資料夾有跟外面的實體做對應，為了要方便拿到檔案，所以放在這邊

## 匯入資料庫

把 dump.zip 中的 sql file 拿出來後放到 Postgres 的 container 中。

> 這邊我對應到 app 的資料夾中

```shell
docker exec -it database bash
su postgres
psql
```

根據官方的文件 [Database Preparation - Docs (gitea.io)](https://docs.gitea.io/en-us/database-prep/#postgresql) 先建立好使用者和資料庫。

```sql
CREATE USER "gitea" WITH PASSWORD 'password';
CREATE DATABASE dbgitea WITH OWNER gitea TEMPLATE template0 ENCODING UTF8 LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8';
```

接著就可以開始匯入資料。

```shell
psql --username gitea -h localhost --set ON_ERROR_STOP=on dbgitea < /app/gitea-db.sql
```

## 修改連線資料

以上的步驟都做完，最後就是要修改 app.ini 裡的連線資訊，找到 `database` 的區塊，把資訊改成以下即可。

```ini
[database]
DB_TYPE  = postgres
HOST     = database:5432
NAME     = dbgitea
USER     = gitea
PASSWD   = password
```

把服務重新啟動，連線進去確認一下所有內容都是遷移前的狀態，那就可以收工啦!!