---
title: '[Debug]Dotnet 9 的 CET 問題'
date: 2024-11-15 20:17:42
updated: 2024-11-15 20:17:42
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
thumbnail: /images/dotnet-core.png
---

在 11/13 的時候 Dotnet 9 正式亮相了，很興奮地馬上跟上，同事們也都裝了，但悲劇馬上發生。

<!-- more -->

安裝 SDK 後同事用 dotnet run 的指令執行，都無法正確的執行，會得到一個錯誤，並且會看到 `project.exe` 的執行錯誤。

> CLR: Assert failure(PID 110152 [0x0001ae48], Thread: 52028 [0xcb3c]): !AreShadowStacksEnabled() || UseSpecialUserModeApc()

我在電腦上執行是非常正常，因此開始了偵錯抓蟲之旅。

先請同事執行 `dotnet build` 確認有出現 `.net9` 的資料夾在 bin 底下，然後再用 `dotnet run project.dll` 確認這樣能執行起來，事實證明沒問題，表示 dotnet 9 sdk 運行是沒問題的，那就是出現在 `dotnet run` 這個指令下。

接著用錯誤去 google，查到 github 有這個 issue [.NET 9 assert failure · Issue #108589 · dotnet/runtime](https://github.com/dotnet/runtime/issues/108589) ，先請同事嘗試在 **csproj** 加上這組設定

```xml
<PropertyGroup>
	<CETCompat>false</CETCompat>
</PropertyGroup>
```

馬上可以正常執行，主要原因是這次有個 [重大變更：默認支援 CET - .NET | Microsoft Learn](https://learn.microsoft.com/zh-tw/dotnet/core/compatibility/interop/9.0/cet-support)，基本上 windows 11 沒有問題，而是 windows 10 會需要更新 [Windows 10 - release information | Microsoft Learn](https://learn.microsoft.com/en-us/windows/release-health/release-information) 才有辦法支援這個功能，剛好同事和我的差異就是 OS。

還好不是大問題，而且是在開發階段才會發生，希望大家能享受這次的 dotnet 版本升級。