---
title: 如何建立 Oracle database image
date: 2023-01-16 10:46:13
categories:
- Database
- Oracle
tags:
- Database
- Oracle
- Docker
thumbnail:
---

原本 Oracle 有提供 image 在 [DockerHub](https://hub.docker.com/_/oracle-database-enterprise-edition)上，現在已經沒有再提供了，那對於習慣用 container 做開發的我來說，非常的不方便，還好 Oracle 也有提供另外的做法可以來建立 image，接下來分享該如何操作。

<!-- more -->

Oracle 已經將很多的服務寫好 dockerfile 放到 [github](https://github.com/oracle/docker-images) 上，從 OracleDatabase 目錄進入，有分 SingleInstance 及 RAC 就根據自己需求去選，這邊我是要自己開發用的所以選擇是 **SingleInstance**，進去以後可以看到從 11g 到 21c 都有提供，就根據自己的需要去選擇，今天會使用 19c 來做為展示。

> 雖然我的環境是用 windows，在 mac 的環境下是沒有差的。

## Sparse checkout

整個 repo 很多內容，我們只需要 SingleInstance 底下的 dockerfiles 資料夾，選擇使用 sparse checkout 的做法，只拿部分的內容。

```shell
git clone --depth 1 --sparse https://github.com/oracle/docker-images.git oracle-images
cd oracle-images
```

設定只取得 `OracleDatabase/SingleInstance/dockerfiles`。

```shell
git sparse-checkout set "OracleDatabase/SingleInstance/dockerfiles"
```

![dockerfiles-folder](dockerfiles-folder.png)

## 下載dbhome

[Database Software Downloads | Oracle](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html) 接著到 Oracle 官網下載 dbhome，選擇好版本並且點擊 linux x86_64 來下載。

![download_dbhome_linux](download_dbhome_linux.png)

下載後的檔案放到剛剛所 clone 的專案目錄底下對應的版本資料夾中。

![dbhome_in_dockerfile_folder](dbhome_in_dockerfile_folder.png)

## 產生image

前置作業終於完成，可以來最作後一個步驟，產生 image 啦!
來到 dockerfiles 的資料夾，看到 `buildContainerImage.sh` 檔案，Oracle 已經寫好指令，讓我們可以更為方便的產生 image。

> 如果是在 windows 環境下，無法直接執行 bash 的檔案，可以改用 WSL 來執行。

後面加上 `-v` 指定版本，`-t` 指定 image tag 名稱。

> 共有三種模式可以選擇，Enterprise Edition(-e), Standard Edition 2(-s), Express Edition(-x)

```shell
./buildContainerImage.sh -v 19.3.0 -s -t oracle/database:19.3.0s
```

執行會有一陣子(我的電腦最後花費 280 秒)，最後看到這樣的訊息就代表成功囉。

```shell
 => => naming to docker.io/oracle/database:19.3.0s                         0.0s

Use 'docker scan' to run Snyk tests against images to find vulnerabilities and learn how to fix them


  Oracle Database container image for 'se2' version 19.3.0 is ready to be extended:

    --> oracle/database:19.3.0s

  Build completed in 280 seconds.
```

開啟 DockerDesktop 切換到 Images 的頁面，就可以看到所產生的 image 啦!!

![image-build-successfully](image-build-successfully.png)

## 執行

最後就是跑起來!! (~~還慢慢走啊，動作~~)
如果想要保留檔案，可以使用 volumn 做對應 `/opt/oracle/oradata`；使用環境變數指定管理者密碼 `ORACLE_PWD`。

> 如果沒有 volumn 的話，第一次的啟動會花較多時間；有 volumn 就會蠻快的。

```shell
docker run --name oracle -d -it --rm -p 1521:1521 -v ${PWD}/data19c:/opt/oracle/oradata -e ORACLE_PWD="p@ssw0rd" oracle/database:19.3.0s
```

到了這個步驟，終於可以開始我們的開發啦，也不用擔心重灌系統還要先 dump/import 資料，可以大大的節省時間啦!!
