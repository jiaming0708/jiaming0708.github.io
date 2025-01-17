---
title: '如何在瀏覽器開啟應用程式'
date: 2021-10-11 10:31:38
updated: 2021-10-11 10:31:38
categories:
- Frontend
tags:
- Frontend
- browser
- Windows
---

現在越來越多的服務會直接用網頁來開啟應用程式（例如 zoom/teams），不管是在手機或是電腦中，這篇主要就是要來分享在 windows 中如何做到這件事情。

<!-- more -->

> 這會是一個系列文，將會介紹如何做安裝程式並且在頁面中自動偵測是否能夠開啟應用程式

我們都知道一般上網的時候都是採用 http 的協定，來瀏覽網頁，如果說你曾經有用過 ftp 這樣的服務，應該記得瀏覽器是能夠直接連 ftp 伺服器。

> http://www.google.com
> ftp://localhost

在前面的 http / ftp 都是屬於一種 URL 協定，那要開啟應用程式，我們也能夠制定這樣的協定，讓系統認識並且啟動我們所需要的指令或程式。

# 註冊應用程式

在 windows 中，**登陸編輯程式** 是一個很強大的功能，我們要讓系統認識新的 URL 協定，就必須透過他來進行。

首先，我們來參考一下 zoom/teams 的作法，開啟 **登陸編輯程式** 後到 `HKEY_CLASSES_ROOT` 底下，找到 `zoommtg` 或者 `ms-teams` 就可以看到像下圖
![teasm](teams.jpg)

嘗試匯出成檔案，來看一下內容有些什麼

```yml
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\ms-teams]
"URL Protocol"=""
@="URL:ms-teams"

[HKEY_CLASSES_ROOT\ms-teams\shell]

[HKEY_CLASSES_ROOT\ms-teams\shell\open]

[HKEY_CLASSES_ROOT\ms-teams\shell\open\command]
@="\"C:\\Users\\Jimmy\\AppData\\Local\\Microsoft\\Teams\\current\\Teams.exe\" \"%1\""
```

最上面的那層機碼就是協定名稱，必須要跟裡面的 URL 相同，並且指定為 URL Protocol，並在 command 指定要啟動的應用程式。
來試著在瀏覽器中輸入 `ms-teams:\\` ，可以看到提示是否開啟 Microsoft Teams。

![launch teams](launch-teams.jpg)

透過上面的範例就可以知道，如何產生一個新的協定 `jimmy-test`，接著我們來試著建立一個新的協定，並且開啟 vscode 這個應用程式。

```yaml
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\jimmy-test]
"URL Protocol"=""
@="URL:jimmy-test"

[HKEY_CLASSES_ROOT\jimmy-test\shell]

[HKEY_CLASSES_ROOT\jimmy-test\shell\open]

[HKEY_CLASSES_ROOT\jimmy-test\shell\open\command]
@="\"D:\\Programes\\Microsoft VS Code\\Code.exe\" \"%1\""
```

儲存成 reg 檔以後執行，並且在瀏覽器中輸入 `jimmy-test://` 就可以看到開啟 vscode 的提示，是不是很簡單呢。

> %1 的值會等於瀏覽器中輸入的值，例如 `jimmy-test://123`，在你的應用程式中，就必須要用這樣的網址做解析

> 如果是在手機的 app ，則是在開發時，就要對外宣告有這樣的協定可以使用，ios/android都有支援，在瀏覽器端作法則是不變

# Reference

* [Launching applications using custom browser protocols – Shotgun Support (shotgunsoftware.com)](https://support.shotgunsoftware.com/hc/en-us/articles/219031308-Launching-applications-using-custom-browser-protocols)
* [使用 URL 開啟 Windows 應用程式](https://blog.poychang.net/registering-and-open-an-app-with-custom-uri-scheme/)