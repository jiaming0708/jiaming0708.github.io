---
title: '[Jenkins] 環境建置 - 整合 Gitea 登入'
date: 2022-03-24 09:20:03
categores:
- CI/CD
tags:
- CI/CD
- Jenkins
- Gitea
---

Jenkins 和 Gitea 的環境都建置好後，會希望登入的使用者能在一個地方控管就好， Gitea 自從 1.8 以後就開始提供 Oauth 的方法給其他系統，今天就針對這個部分來做介紹。

<!-- more -->

# 設定 Gitea

進入系統後，開啟 `設定` > `應用程式`，拉到最下面的 `管理 OAuth2 應用程式` 區塊。

- 應用程式名稱: `Jenkins`
- 重新導向 URI: `http://{domain}/securityRealm/finishLogin`

輸入以上資訊後按下 `建立應用程式`。

![gitea-oauth-create](./gitea-oauth-create.png)

就會拿到一組 `客戶端 ID` 及 `密鑰` ，這個頁面先不要關閉，等一下要把這兩個資訊貼到 Jenkins 上。

> 建議使用公用帳號操作，避免使用者被砍掉或停用，導致後續的麻煩

# 設定 Jenkins

## 安裝 Plugin

Jenkins 的 OAuth 相關 plugin 蠻多的，但就是沒有直接對應 Gitea 的，可以選擇這套 [OpenId Connect Authentication | Jenkins plugin](https://plugins.jenkins.io/oic-auth/) 來作為整合的套件。
進入 `Manage Jenkins` > `Plugin Manager` > `Available`，安裝套件並且重啟。

## 設定 Oauth

安裝完套件後，進入到 `Manage Jenkins` > `Confiure Global Security`，找到 `Security Realm` 區塊

![jenkins-security-realm](./jenkins-security-realm.png)

預設應該會使用 `Jenkins’ own user database` ，要改成 `Login with Openid Connect`，並且將前面 Gitea 所產生 `客戶端 ID` 及 `密鑰` 填入，`Configuration mode` 選擇 `Automatic confiuration` 填入 Gitea 的網址。

![jenkins-login-openid-connect](./jenkins-login-openid-connect.png)

設定完成以後，登出系統，並且回到 Jenkins 網站的跟目錄，就會發現被轉到 Gitea 網站做登入授權。

![gitea-auth-application](./gitea-auth-application.png)

# 結論

其實跟 DroneCI 很像，可以參考 {% post_link gitea-drone-docker %} 的作法，雖然沒有直接的套件讓我們整合 Gitea，但也是能夠做到相同的事情。
