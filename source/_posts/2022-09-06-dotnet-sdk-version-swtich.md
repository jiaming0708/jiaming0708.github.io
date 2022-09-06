---
title: '[Dotnet] 如何切換 SDK 版本'
date: 2022-09-06 15:00:16
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
- SDK
---

DotnetCore的版本更新的很快，為了要有支援多個環境，電腦都會安裝很多的 SDK，預設 cli 會使用最新的版本，理論上新版都應該是可以支援舊版的編譯；但偏偏~~雨漸漸~~為了要測試一個功能，就安裝了 Dotnet 7 preview，結果導致原本可以編譯的專案變成失敗，而且錯誤訊息還很難看懂，還好可以指定舊版的 SDK，就可以避免掉這樣的問題。

<!-- more -->

首先我們要先確定現在電腦中有安裝那些版本，在 cli 中下指令來取得清單，裡面包含版本號及安裝路徑。

```shell
dotnet --list-sdks
```



![sdk-list](sdk-list.png)

接著可以透過 cli 來產生 `global.json` 的檔案，用來指定 SDK 版本。

```shell
dotnet new globaljson
```

![cli-globaljson](cli-globaljson.png)

要注意的是，這個檔案會產生在你所下的目錄位置內，並不是真的在 User 底下的一個檔案，可以透過 cat 來看內容是什麼。

![cat-globaljson](cat-globaljson.png)

像是我要改成指定 `6.0.300` 的話，就可以開啟這個檔案，並且替換 version 的內容，存檔後可以下 `dotnet --version` 來確認現在版本是符合預期的。

> 如果只想要透過指令，也可以考慮使用 jq 這個套件來完成，或是用 vim

![change-sdk-version](change-sdk-version.png)

當然也有更快的方法，產生 global.json 時就直接指定需要的版本號。

```shell
dotnet new globaljson --sdk-version 6.0.300
```

![cli-globaljson-version](cli-globaljson-version.png)

用以上的方法就可以先排除掉因為 SDK 太新導致編譯失敗的問題，不建議把這個檔案加入版控，畢竟這算是個人環境因素，另外就是如果未來專案升級，這個檔案也要砍掉或跟著調整，不然執行上就會有問題。

# Reference

- [global.json overview - .NET CLI | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/core/tools/global-json?WT.mc_id=DT-MVP-4015686)
- [如何在多個 .NET Core SDK 版本之間進行切換 (global.json) | The Will Will Web (miniasp.com)](https://blog.miniasp.com/post/2018/04/19/How-to-switch-between-DotNet-SDK-versions)