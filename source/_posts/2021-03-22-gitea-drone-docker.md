---
title: 在本機使用Docker架Gitea和DroneCI
date: 2021-03-22 10:04:38
categories:
- CI/CD
tags:
- CI/CD
- Gitea
- Drone
- Docker
---

當要選擇自架的 git 伺服器，Gitea 是蠻多公司的選擇，在這個講求效率的開發時代，必須要搭配一個 CI/CD 的平台，讓我們可以更好更快的部屬，我選擇 Drone CI 作為平台，今天要來分享一下使用 docker-compose 建置在本機的步驟。

<!-- more -->

# Gitea

跟著[官方的文件](https://docs.gitea.io/en-us/install-with-docker/)試著把 Gitea 的服務建立起來，在這邊就只弄最簡單的SQLite，首先要建立一個 `docker-compose.yml` 的檔案，並且將內容貼上。

```yaml
version: "3"

networks:
  gitea:
    external: false

services:
  gitea-server:
    image: gitea/gitea:1.13.3
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
    ports:
      - "3000:3000"
      - "222:22"
```

試著跑看看 `docker-compose up -d` ，成功的話就可以看到 **done** 的字樣。

```shell
> docker-compose up -d
Creating network "test_gitea" with the default driver
Creating gitea ... done
```

再用瀏覽器打開 `localhost:3000` 看到以下畫面就沒問題了！

![gitea install](gitea-install.png)

# Drone

Drone 可以跟很多平台做整合，這邊當然就是選擇前面已經架好的 Gitea 作為我們的目標 [參考文件](https://docs.drone.io/server/provider/gitea/)。Drone 總共要起兩個服務，一個是主服務，另一個是 runner 專門執行 task 的。

## Gitea OAuth

根據文件，第一步要在 Gitea 的設定中找到 `Applications` 這個頁籤，建立一個新的應用。名稱我們給他 `drone`，URI 的話預期是 `http://locahost:8000`。

> 這邊要特別注意的是 `Redirect URI` 最後必須要帶上 **/login**，不然會無法成功

![Gitea create oauth application](gitea-create-oauth-application.png)

按下建立以後就可以看到以下的畫面，這個畫面先留著不要馬上關掉，或者是把 `Client ID` 以及 `Client Secret` 存起來，等一下設定會用到。

![gitea application](gitea-application-data.png)

## 產生共用密碼

接著我們要產生一個共用密碼，給主服務和 runner 之間做認證使用。

```shell
> openssl rand -hex 16
81e04d83a6054b464f5c5b13365578fd
```

## 設定docker-compose

為了讓 container 之間能夠互相的溝通，加上前面的 gitea 已經使用 compose，這邊就繼續往下編輯，讓我們使用一個檔案來起多個服務。

將資料填入對應的欄位，並且儲存 `docker-compose.yml` 。

```yaml
  gitea-server:
    # ...
  drone-server:
    image: drone/drone:1
    container_name: drone-server
    ports:
      - 8000:80
    volumes:
      - ./drone:/data
    restart: always
    networks:
      - gitea
    depends_on:
      - gitea-server
    environment:
      - DRONE_SERVER_HOST=localhost:8000 # drone server
      - DRONE_SERVER_PROTO=http
      - DRONE_RPC_SECRET=81e04d83a6054b464f5c5b13365578fd #共用密碼
      # Gitea
      - DRONE_GITEA_CLIENT_ID=cf9e05e8-53b0-447b-a8d7-86d7c882c64a # client id
      - DRONE_GITEA_CLIENT_SECRET=6nZ8EEJtk6lqI00Fdbf4A3F7_2-rSzxbQbikw-EnsvU= #client secret
      - DRONE_GITEA_SERVER=http://localhost:3000 # gitea server
```

執行跟剛剛一樣的指令 `docker-compose up -d` ，不需要先把 gitea 停掉，就可以啟動 drone。

```sh
> docker-compose up -d
gitea is up-to-date
Creating drone-server ... done
```

好了！來用瀏覽器開啟 `localhost:8000` 這個服務，看到這個畫面表示成功啦！！

![drone authorize](drone-authorize.png)

## 錯誤

auth 的按鈕按下去以後，會看到一個錯誤訊息。

> Login Failed. Post "http://localhost:3000/login/oauth/access_token": dial tcp 127.0.0.1:3000: connect: connection refused

經過 google 大神說明，原來是 container 的網路都是獨立，所以不接受 127.0.0.1，要使用 192.168.0.x 或者是 domain 的方式。

# ngrok

ngrok 是一個可以將本機的服務直接轉成對外的服務，很適合還在開發階段，但又要給其他人確認畫面之類的時候。因為剛剛的 drone 不能使用 localhost 作設定，因此我們可以先使用這個服務來做測試，確認整個設定都是正確。

## 設定

這邊我們要把預計使用到的 3000 以及 8000 兩個 port 都做監聽，ngrok 的指令一次只能起一個 port ，因此我們要透過 `ngrok.yml` 做設定一次起多個。

```yaml
authtoken: {{your token}}
tunnels:
  gitea:
    addr: 3000
    proto: http
  drone:
    addr: 8000
    proto: http
```

執行 `./ngrok start --all --config ngrok.yml`，應該可以看到底下的畫面。

![ngrok](ngrok.png)

## 修改 docker-compose.yml

將 ngrok 所產生的 domain 貼回去 `docker-compose.yml` 中，這邊只有兩個地方要調整

- DRONE_SERVER_HOST
- DRONE_GITEA_SERVER

```yaml
  drone-server:
    image: drone/drone:1
    container_name: drone-server
    ports:
      - 8000:80
    volumes:
      - ./drone:/data
    restart: always
    networks:
      - gitea
    depends_on:
      - gitea-server
    environment:
      - DRONE_SERVER_HOST=b42812f5966a.ngrok.io #改用nrok的domain
      - DRONE_SERVER_PROTO=http
      - DRONE_RPC_SECRET=81e04d83a6054b464f5c5b13365578fd
      # Gitea
      - DRONE_GITEA_CLIENT_ID=cf9e05e8-53b0-447b-a8d7-86d7c882c64a
      - DRONE_GITEA_CLIENT_SECRET=6nZ8EEJtk6lqI00Fdbf4A3F7_2-rSzxbQbikw-EnsvU=
      - DRONE_GITEA_SERVER=http://4fe6164979a9.ngrok.io #改用nrok的domain
```

## 修改 Application 的 redirect URI

重新啟動服務以後，一樣可以使用 localhost:3000 進去 gitea 的服務頁面，將前面設定好的 `Redirect URI` 修改為 ngrok 的 domain。

都修改好後，就可以重新進入 8000 的那個 domain，應該就可以成功進入到 drone 的首頁。

![drone empty home](drone-empty-home.png)

# 測試

## drone runner

前面有講到 drone 是分兩個服務，一個 server 另一個是 runner，但剛剛其實都沒有起這個服務，在測試之前，先將這個服務起起來！

```yaml
  drone-agent:
    image: drone/drone-runner-docker:1
    container_name: drone-runner
    restart: always
    depends_on:
      - drone-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_RPC_HOST=b42812f5966a.ngrok.io
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_SECRET=81e04d83a6054b464f5c5b13365578fd
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=${HOSTNAME}
```

## push 一個 commit做測試

萬事具備！先建立一個 repo 並且 push 檔案 `.drone.yml` 作為確認。

```yaml
kind: pipeline
type: docker
name: default
   
steps:
- name: test
  image: alpine
  commands:
  - echo hello
  - echo world
```

可以看到一個 task 就跑起來了！但是，凡事都有個 but！在 clone 的時候竟然失敗了！！！

![drone pull fail](drone-pull-fail.png)

## 重新設定gitea

還記得一開始啟動 gitea 的時候，我們什麼都沒有改，直接就建立了，但後來我們使用了 ngrok 作為 domain，在 gitea 建立後這個 URI 就已經被固定，因此我們要在 `docker-compose.yml` 中加上 `ROOT_URL` ，並且將 gitea 的資料夾砍掉，重新啟動！

> 這邊可以使用 `docker-compose down` 的指令先將服務都停止

```yaml
  gitea-server:
    image: gitea/gitea:1.13.3
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - ROOT_URL=http://4fe6164979a9.ngrok.io
    restart: always
    networks:
      - gitea
    volumes:
      - ./gitea:/data
    ports:
      - "3000:3000"
      - "222:22"
```

在 install 的設定中找到 `Gitea Base URL` 這個欄位，並且確認 domain 是預期的 ngrok。

![gitea base url](gitea-base-url.png)

接著按照前面的步驟，重新建立 application 等的動作，就可以看到 drone 成功的跑完啦！！

![drone run successful](drone-sucessful.png)

# 結論

這些服務在一開始設計的時候，就不是預期在本機，如果是在有 domain 的 server 上，這些問題都不會發生。

> 很妙的是 drone 用 0.8 就可以使用 localhost，應該是後面他們有做過調整導致。

