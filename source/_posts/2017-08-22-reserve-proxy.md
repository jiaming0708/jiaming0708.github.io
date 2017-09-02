---
title: 設定Reserve Proxy在Windows、Mac
date: 2017-08-22 10:26:37
categories:
- Back-end
- IIS
tags:
- IIS
---

# 前言

因為測試的需要，protrator-recorder是在local環境中架設走的是http，但公司的測試環境是在https底下，因此會導致無法將protrator-recorder掛載到測試環境中進行錄製，就開始找方法去解決

錯誤訊息是mixed content，讓我一開始找的方向就是把工具架成https，錄製工具總共要起兩個網站，第一個是webdriver，第二個是recorder本身

第一個想法就是針對recorder本身改成https，原生程式是用node的express+http，只要改成https即可，但是問題來了，webdriver這個工具該怎麼架成https，這部份我沒找到資料(如果有人知道麻煩告訴我)，在經過幾番詢問後，kevin給了一個新的方向 `reverse proxy` 

# Reverse Proxy

在開始之前，要先複習一下一般的Proxy功能(也就是Forward Proxy)

透過proxy可以直接連線到無法直接連線的網站，還蠻常應用在要連國外的網站只允許自己國家內的ip；或者是在公司內部不允許對外，就必須要透過proxy來做對外的接口

![proxy-server](proxy-server.png)

複習完Proxy後，我們要來看看什麼是Reverse Proxy

從字面上來看就是反向，我們連線到一個網站，但是他透過ReverseProxy去連線到背後真實的網站，那對使用者來說其實沒有影響，因為還是可以連到對的網站進行瀏覽

![reverse-proxy](reverse-proxy.jpg)

# 架設

本身有兩個開發環境，mac、windows，底下就針對這兩個環境下怎麼架設來說明一下

## Mac

在mac環境下可以使用 [mitmdump](http://docs.mitmproxy.org/en/stable/index.html) ，照著官方安裝即可

安裝好後，要設定reverse proxy，可以參考[這篇](http://docs.mitmproxy.org/en/stable/features/reverseproxy.html)，只要在後面加上-r並且給目標https網站即可
那我們瀏覽的時候只要連`http://localhost`即可

> mitmdump -R https://httpbin.org -p 80

## Windows

在mac所用的mitmdump這套軟體，很可惜的在windows只剩下UI版本，沒有command mode，那我們所需要reverse proxy就不知道該怎麼設定，因此要改其他方法

還好強大的微軟(XDDD)在IIS已經先埋好這個功能，要在IIS上使用，必須要先安裝兩個套件

* [ARR(Application Request Routing)](https://www.iis.net/downloads/microsoft/application-request-routing)
* [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite)

> 系統環境為:(<font color="red">其他環境的操作畫面可能不同!</font>)
> Win 10
> IIS 10

安裝完畢後，先進行ARR的設定，將Proxy啟用

![IIS_ARR](/IIS_ARR.png)

![ARR_main](/ARR_main.png)

![ARR1](/ARR1.png)

![ARR2](/ARR2.png)

接著就可以設定URL Rewrite的設定

![IIS_url_rewrite](/IIS_url_rewrite.png)

![URL_Rewrite_main](/URL_Rewrite_main.png)

![URL_Rewrite_rule](/URL_Rewrite_rule.png)

![URLrewrite1](/URLrewrite1.png)

![URLrewrite2](/URLrewrite2.png)

> 這邊要注意一下，重寫URL後面帶的參數，如果錯誤就無法正確導入
> 可以使用測試模式，確認一下內容

# 參考

[Microsoft - Reverse Proxy with URL Rewrite v2 and Application Request Routing](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/reverse-proxy-with-url-rewrite-v2-and-application-request-routing)

[Microsoft - Creating Rewrite Rules for the URL Rewrite Module](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/creating-rewrite-rules-for-the-url-rewrite-module)

[Reverse Proxy](https://en.wikipedia.org/wiki/Reverse_proxy)