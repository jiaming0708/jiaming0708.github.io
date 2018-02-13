---
title: docker
date: 2018-01-08 20:46:53
categories:
- Docker
tags:
- Docker
---

# Why

身為開發者的你也許常常碰到以下的情況

* 每個人的電腦環境都不同，常常說`在我電腦上是好的`，但在production或是同事的環境就是有問題
* 新同事進來要重新setup一個全新的環境，然後又跟其他人不同
* mac、window跑起來不同
* 模擬環境非常的困難

其實簡單講起來，就是因為環境的不同，很容易造成偵錯或是開發上的困擾，要解決這個問題，最簡單的作法就是把環境統一，這樣就不會那麼多的理由和藉口可以去說嘴

在以往可能會選擇使用VM來解決這個問題，但是一個vm動輒好幾G，常常光要複製就是一個困擾，而且開發上也用不到很多東西，因此我們需要更輕量的一個解決方案`docker`

# What

簡單說就是虛擬機器，進階一點的說是輕量的虛擬機器；與傳統VM有什麼不同呢，可以透過下圖來做檢視，container少了一層，交由docker來實作這個部分

![virtual](virtualization.png)
![docker](docker.png)

# How

dockerg是由container、image這兩個組合起來的變形金剛（誤），那接著來介紹一下這兩個玩意

## image

透過指令將所需要的環境先寫好，container就可以直接使用這些所寫好的image來執行

##  container

用來裝載image，並且每個container是獨立的不會互相影響

* 不可變
* 輕量
* 快速
* 一次性
* 指令式(CI!!)

# Use

就讓我們開始進入docker的世界吧！

## dockerfile

除了可以直接上[docker hub](https://hub.docker.com/)下載以外，當然也可以自己產生image（自己的內容自己做）
指定基底的image，再來就是開始安裝我們需要的東西

```dockerfile
FROM node:8
# 執行安裝
RUN yarn global add @angular/cli
RUN ng -v
# 指定工作路徑
WORKDIR /app
# 複製檔案到container
ADD ./src /app
```

# command

docker是用go語言基於linux系統開發出來，所以就是滿滿的指令，而且寫的時候也要對image的環境指令有一定熟悉度，弄起來才不會一直碰壁...(苦主QQ)

### build

寫好的dockerfile產生成為image；產生一個叫做`angular`的image

> docker build -t angular .

### run

將container啟動並且執行指令；指定本機4000的port連接到4200，並且執行ng serve的指令

> docker run -p 4000:4200 angular ng serve

# docker compose

可以定義與執行多個container，只要用簡單的指令，來完成全部的事情
(懶人專用...把指令都寫好只要呼叫就好)

> docker-compose up

## yaml

透過`docker-compose.yml`可以事先設定好上面所輸入的那些docker指令

```yaml
version: '3'
services:
  dev:
    build: .
    ports:
      - "4200:4200"
    command: ng serve -host 0.0.0.0
  prod:
    build: .
    ports:
      - "80:4200"
    command: ng serve -host 0.0.0.0 --prod
```

# conclusion

docker是一個hen方便的工具，除了讓你快速複製環境，還能跟CI做個搭配，資源還不會吃太多，一起跳坑吧！