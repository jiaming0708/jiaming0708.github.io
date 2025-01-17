---
title: '[nginx] reverse proxy 在 docker compose 架構下'
date: 2022-06-22 19:46:46
updated: 2022-06-22 19:46:46
categories:
- nginx
tags:
- nginx
- Docker
- reverse-proxy
- ngrok
---

在非 windows 的環境下架站，nginx 會是首選，把一些服務搭配 nginx 並且使用 docker compose 作架站，也是一個不錯的選擇。
今天會使用 Gitea/Drone 這兩個服務，並且搭配 ngrok 來作為模擬。

<!-- more -->

# 前言

之前寫過另一篇 {% post_link gitea-drone-docker %}，用 ngrok 對應 3000 和 8000 兩個 port，如果是用 ngrok 還算方便，畢竟都還是 80/443 的連線，但如果轉換成自己的 domain，就會變成 `git.jimmy.com:3000` 和 `ci.jimmy.com:8000` 這樣的連線方式，對於這樣的網址其實是難以接受的，那我們可以使用 reverse proxy 來達成。

之前我也是用 IIS 做 reverse proxy，gitea 相關的設定可以參考這兩篇 {% post_link iis-reverse-proxy-header %} 和 {% post_link gitea-iis-reverse-proxy-trouble-shooting %}，如果你是用 windows 10 來作為伺服器，就會遇到一個問題同時連線數過多時就會連不上的問題，這個在微軟的文件中有提到 [Understanding IIS Request Restrictions on Windows Client OS | Microsoft Docs](https://docs.microsoft.com/en-us/iis/troubleshoot/request-restrictions)，改用 nginx 就可以忽略掉這個問題。

好啦~囉嗦完後還是要實際看看怎麼樣的實作吧!

# 設定

使用 ngrok 作為本機架站的工具，並把 gitea/drone 兩個服務躲在 nginx 背後，只開放 80 port 出來。

## ngrok

首先針對 `ngrok.yml` 做調整，改成設定兩個都是 80 port。

> 也許你會擔心不能跑，這點倒是不用擔心。

```yaml
authtoken: {{your token}}
tunnels:
  gitea:
    addr: 80
    proto: http
  drone:
    addr: 80
    proto: http
```

執行 `ngrok.exe start --all --config ngrok.yml`，就可以看到以下的畫面，會有兩個 domain 都對應著 80。

![ngrok](ngrok.png)

## docker-compose

先把 domain 搞定以後，接著要來調整 `docker-compose.yml` 的內容，把 gitea 和 drone 對外的 port 都刪掉，改加上 nginx 並且開放 80 port 對外。

```yaml
version: "3.9"

services:
  gitea:
    image: gitea/gitea:1.16
    container_name: gitea
    environment:
      - ROOT_URL=http://1374-1-171-251-189.ngrok.io
      - DEFAULT_BRANCH=main
    restart: always
    volumes:
      - ./gitea:/data
  drone-server:
    image: drone/drone:2
    container_name: drone-server
    volumes:
      - ./drone:/data
    restart: always
    environment:
      - DRONE_SERVER_HOST=8372-1-171-251-189.ngrok.io
      - DRONE_SERVER_PROTO=http
      - DRONE_RPC_SECRET=d733ce6f00905a05395ea8198c22234b
      # Gitea
      - DRONE_GITEA_CLIENT_ID=12d97e72-1489-4df6-a547-946ded390617
      - DRONE_GITEA_CLIENT_SECRET=e7lAy3SbWAKDm5FaVIuBr6vfQQIVLXUCitkv4xLFIQIP
      - DRONE_GITEA_SERVER=http://1374-1-171-251-189.ngrok.io 
      - DRONE_GIT_ALWAYS_AUTH=true
  nginx:
    image: nginx
    restart: always
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - gitea
      - drone-server
networks: 
  default: 
    name: infra
```

## nginx

在 docker compose 文件 [Networking in Compose | Docker Documentation](https://docs.docker.com/compose/networking/) 中提到，**同一個 network 裡面**，可以直接網內互打，不用知道另一個 container 的 ip，只要指定名稱就可以，我們就可以透過這個功能，讓 nginx 直接對應到 service。

底下是 `nginx.conf` 的設定。

```nginx
http {
    server {
        listen 80;
        listen [::]:80;
        server_name 1374-1-171-251-189.ngrok.io;
        
        location / {
            proxy_pass http://gitea:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

    server {
        listen 80;
        listen [::]:80;

        server_name 8372-1-171-251-189.ngrok.io;

        location / {
            proxy_pass http://drone-server:80;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Host $host;
            proxy_redirect off;
            proxy_http_version 1.1;
            proxy_buffering off;
            chunked_transfer_encoding off;
        }
    }
}
```

# Reference

- [Usage: Reverse Proxies - Docs (gitea.io)](https://docs.gitea.io/en-us/reverse-proxies/)
- [Documentation for Drone behind SSL reverse proxy - Support - Drone](https://discourse.drone.io/t/documentation-for-drone-behind-ssl-reverse-proxy/3373)
- [Networking in Compose | Docker Documentation](https://docs.docker.com/compose/networking/)