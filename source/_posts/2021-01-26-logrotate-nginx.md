---
title: 設定nginx的log備份機制
date: 2021-01-26 10:25:27
updated: 2025-01-22 09:14:27
categories:
- linux
tags:
- nginx
---

前幾天公司部屬新的版本到 prod server 的時候失敗了，檢查以後才發現是空間不足，問題是出現在 nginx log 目錄太肥，到底是發生什麼事情，今天來探討一下。

<!-- more -->

# 環境與前言

目前我們還有些服務還是直接架設在 ec2 上，所以我們會在上面安裝 nginx 來指向我們的服務，因為這台 server 已經運行蠻久的時間，上面的版本也比較舊，為了安全性問題，前陣子將 nginx 升級到 1.18（強烈建議安裝到此版本，之前的版本都有安全性的問題），但升級以後沒有注意到 logrotate 的設定跟之前不同。

- ubuntu v16
- nginx v1.18

# logrotate

`logrotate` 是一個 linux 上的定時服務，會根據設定來做壓縮以及產生新的檔案。

> 舉例來說，log是以天為單位作為切分，那 logfile.1 就會變成 logfile.2 並且產生新的 logfile.1

## 目前設定

執行以下指令查看 logrotate 目前設定。

> cat /etc/logrotate.d/nginx

```
/var/log/nginx/*.log {
        daily
        missingok
        rotate 52
        compress
        delaycompress
        notifempty
        create 640 nginx adm
        sharedscripts
        postrotate
                if [ -f /var/run/nginx.pid ]; then
                        kill -USR1 `cat /var/run/nginx.pid`
                fi
        endscript
}
```

## 參數介紹

* daily，以天為單位做檔案切割，另外還有其他三種
  * weekly
  * monthly
  * yearly
* missingok，沒檔案的時候略過且不會發生錯誤
* rotate 52，保留的檔案個數，這邊指的是52個檔案
* compress，切割出來的檔案，將會壓縮成 gzip
* delaycompress，延遲壓縮，避免檔案還被咬著導致失敗
* notifempty，檔案內容是空的時候不做動作
* create 640 nginx adm，產生檔案時要給的使用者及權限
* sharedscripts，一般來說 postrotate / prerotate 會在每個檔案 rotate 後執行，加上這個指令的話就等於只會執行一次
* postrotate，在 endscript 的區間內是 rotate 結束後會執行的指令

## 調整設定

根據上面的一些介紹，可以得知會保留52天的檔案，並且一個檔案沒有上限，那為了讓開檔更順利且不會佔用太多的空間，限縮檔案是 `10M` 並且只保留 `7個`。

```
--- /etc/logrotate.d/nginx
+++ /etc/logrotate.d/nginx
/var/log/nginx/*.log {
+	size 10M
        daily
        missingok
-	rotate 52
+       rotate 7
        compress
        delaycompress
        notifempty
        create 640 nginx adm
        sharedscripts
        postrotate
                if [ -f /var/run/nginx.pid ]; then
                        kill -USR1 `cat /var/run/nginx.pid`
                fi
        endscript
}
```

## 測試執行

基本上 logrotate 是每天定時跑起來，改完以後是可以不用特別重新執行，但通常改完我們會希望確定是不是如我們預期，那我們可以下個指令要求立刻執行。

> sudo logrotate -v /etc/logrotate.d/nginx

# Reference

* [StatckExchange - How to make log-rotate change take effect](https://unix.stackexchange.com/questions/116136/how-to-make-log-rotate-change-take-effect/116141#116141)

* [Linux man page - logrotate](https://linux.die.net/man/8/logrotate)

