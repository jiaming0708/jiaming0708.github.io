---
title: '[dotnet] 如何自訂義驗證行為'
date: 2023-01-18 16:12:19
categories:
- Backend
- API
tags:
- Dotnet
- Authorization
thumbnail: /images/dotnet-core.png
---

在 dotnet MVC 的專案架構中，要做權限驗證的話很輕鬆， dotnet 都已經幫我們整合好，簡單一點的幾行程式就可以。
剛好有一個需求是希望能夠針對某些 API 要檢查使用者的一些權限，這些資訊比較多又加上使用者可以隨時調整權限，開始思考有什麼方法可以不要每隻重寫又可以做到驗證這件事情，剛好看到官方文件中有一篇說明該如何實作，這邊就來介紹一下。

<!-- more -->



## Reference

- [Policy-based authorization in ASP.NET Core - Learn Microsoft](https://learn.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-7.0)
- [ASP.NET Core 6 使用 Policy Authorization 保護端點 - 余小章 @ 大內殿堂](
