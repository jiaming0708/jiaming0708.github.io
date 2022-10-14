---
title: '[dotnet framework] 如何使用 cli 執行測試'
date: 2022-10-14 14:27:37
categories:
- Backend
- Dotnet
tags:
- Dotnet
- Testing
thumbnail: /images/dotnet-framework.jpg
---

昨天收到同事的一個 bug，使用公司 library 所提供的方法所引發的一個錯誤，這錯誤蠻基本也蠻蝦的，雖然這個 library 已經很少再改，基於保護自己的想法，還是要幫這個方法加上測試，因為這個專案的版本還是 dotnet framework 4.5.1，希望能透過 cli 的方式在 CI 的時候跑測試。

在 cli 測試的話目前有兩種做法，第一種是新版 SDK 所提供的 `dotnet cli`；另一種就是 Visual Studio 的工具。

<!-- more -->

## 使用 VisualStudio 的工具

VS 所提供的工具其實有兩種，一個是舊的作法 `mstest.exe` ，另一才是 `vstest.console.exe`。

> 不管用哪一種方法，都要先編譯專案產生 dll 後才能執行。 

### MSTest.exe

**微軟已經不建議使用這個做法**，找到安裝 VS 的目錄底下 `Common7\IDE\MSTest.exe` 

> {install folder}\Microsoft Visual Studio\\{version}\\{edition}\Common7\IDE\MSTest.exe

示範的指令如下

```shell
MSTest.exe /testcontainer:testproject\bin\debug\testproject.dll
```

更詳細的指令介紹請參考 [使用 MSTest.exe 指令來進行測試 - Yowko's Notes](https://blog.yowko.com/mstest-exe/)

### VSTest.Console.exe

微軟所提供比較新的做法，官方有詳細的介紹 [VSTest.Console.exe command-line options - Visual Studio (Windows) | Microsoft Learn](https://learn.microsoft.com/en-us/visualstudio/test/vstest-console-options?view=vs-2022)。

最後我找到的檔案目錄和官方介紹的不一樣，大家可以都試試看。

- 官方檔案路徑

> {install folder}\Microsoft Visual Studio\\{version}\\{edition}\common7\ide\CommonExtensions\\{Platform | Microsoft}

- 我的檔案路徑

> {install folder}\Microsoft Visual Studio\\\{version}\\{edition}\Common7\IDE\Extensions\TestPlatform

示範的指令如下

```shell
vstest.console.exe testproject\bin\debug\testproject.dll
```

> 若要指定執行的環境為 x64，可以加上參數 `/platform:x64`

## 使用 dotnet cli

自從微軟推出 dotnet core 以後，也跟著推出 dotnet cli，理論上你可以使用 cli 來執行各種專案(包含 framework)，但我自己測試後發現 dotnet build/msbuild 對於 framework 的編譯會有些錯誤，後面就沒細去看。
回到這篇的主軸，可以使用 dotnet test 這個指令，來針對 framework 版本的專案進行測試，但直接執行的話什麼結果都沒，應該是專案的結構導致 dotnet cli 看不懂。

```shell
❯  dotnet test
  正在判斷要還原的專案...
  所有專案都在最新狀態，可進行還原。
```

仔細看一下[官方的文件](https://learn.microsoft.com/zh-tw/dotnet/core/tools/dotnet-test)，可以針對以下幾種來執行 [\<PROJECT> | \<SOLUTION> | \<DIRECTORY> | \<DLL> | \<EXE>]，既然 framework 的結構讓 dotnet cli 看不懂，就跟前面一樣針對 dll 來進行測試。

```shell
❯  dotnet test testproject\bin\Debug\testproject.dll
Microsoft (R) Test Execution Command Line Tool 17.3.1 (x64) 版
Copyright (C) Microsoft Corporation. 著作權所有，並保留一切權利。

正在啟動測試執行，請稍候...
總共有 1 個測試檔案與指定的模式相符。

已通過! - 失敗:     0，通過:     2，略過:     0，總計:     2，持續時間: 596 ms - testproject.dll (net451)
```

## 結論

dotnet framework 畢竟是舊時代的產物，如果要使用新時代的東西 dotnet cli，就要稍微繞路一下，但這個前提就會是 CI server 有 dotnet cli，如果沒有的話，可以直接採用 vstest.console.exe，應該會比較好處理。

## Reference

- [使用 MSTest.exe 指令來進行測試 - Yowko's Notes](https://blog.yowko.com/mstest-exe/)
- [VSTest.Console.exe command-line options - Visual Studio (Windows) | Microsoft Learn](https://learn.microsoft.com/en-us/visualstudio/test/vstest-console-options?view=vs-2022)
- [dotnet test 命令 - .NET CLI | Microsoft Learn](https://learn.microsoft.com/zh-tw/dotnet/core/tools/dotnet-test)