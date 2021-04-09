---
title: '[IIS] 修正 reverse proxy 的 host 問題'
date: 2021-04-09 11:36:52
categories:
- IIS
tags:
- IIS
---

使用 IIS 做 reverse proxy 時，如果網站需要 OAuth，會自動導去其他網站，但會發現這時候 host 沒有被切過去，導致整個 auth 失敗，來分享一下可以怎麼樣解決這個問題。

<!-- more -->

# 問題描述

先把問題描述的清楚一點，如果你的情況剛好跟我一樣，就可以參考參考。

在 window 的 server 上，架了兩個服務，要透過 IIS 設定 domain 對應到這兩個服務，使用 URL rewrite 這個模組中的反向代理 (reverse proxy) 設定。

> 這邊就不介紹怎麼設定 reverse proxy，最下面會放相關連結可以參考。

* git.com -> 3000
* ci.com -> 8000

當進入 ci 這個 domain 會先轉到 git 這邊做 OAuth 的認證，預期的流程如下

1. 在瀏覽器輸入 ci.com
2. 轉導至 git.com/login?redirect=ci.company.com
3. 帳號登入，導回 ci.com

那問題出現在第二步，host 沒有轉過去，還保留在 ci 這個 domain ，除了 host 以外，其他路徑及參數都是正確。

> 錯誤的 URL, ci.com/login?redirect=ci.com

# 解決方法

將 ARR 中的 **Reverse rewrite host in response headers** 關閉即可。

在 IIS 的 root 找到 `Application Request Routing(ARR)`

![iis home](iis-home.png)

在 ARR 的右側點選 `Server Proxy Setting`

![arr home](arr-home.png)

找到 `Reverse rewrite host in response headers` 並且取消核選

![arr setting](arr-setting.png)

# 參考

* {% post_link reserve-proxy %}
* [如何利用 IIS7 的 ARR 模組實做 Reverse Proxy 機制](https://blog.miniasp.com/post/2009/04/13/Using-ARR-to-implement-Reverse-Proxy)
* [URL Rewrite keeps original host Location when reverse proxy 301 redirects](https://stackoverflow.com/questions/23508938/url-rewrite-keeps-original-host-location-when-reverse-proxy-301-redirects)
* [The OAuth Login and Approve has the wrong URL when using IIS as a proxy](https://confluence.atlassian.com/kb/the-oauth-login-and-approve-has-the-wrong-url-when-using-iis-as-a-proxy-540082192.html)