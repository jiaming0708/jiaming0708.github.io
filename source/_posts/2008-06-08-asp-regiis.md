---
title: '[C#] 使用非ASP.NET 1.1'
date: 2008-06-08 10:29:17
updated: 2008-06-08 10:29:17
categories:
- Backend
- ASP.NET
tags:
- ASP.NET
---
指定的Web伺服器不是執行ASP.NET 1.1版。您將無法執行ASP.NET Web應用程式或服務。

<!--more-->

因為IIS跟.FrameWork安裝順序錯誤，導致IIS無法辨別ASP.NET 1.1，所以必須讓兩者
認識得重新安裝ASP.NET 1.1 方法為

開啟「命令提示字元」視窗(按一下 [開始]、[執行]，鍵入 cmd，然後再按一下 [確定])。

瀏覽至您要使用的 Aspnet_regiis.exe 版本的目錄。請注意，.NET Framework 的每一
個版本都附帶有自己的版本。這個檔案通常位於下列目錄： （可使用「檔案總管」瀏覽，
查明路徑）

systemroot\Microsoft.NET\Framework\versionNumber
（例：C:\WINDOWS\Microsoft.NET\Framework\v1.1.4322
於「命令提示字元」視窗命令提示列輸入及執行
cd C:\WINDOWS\Microsoft.NET\Framework\v1.1.4322
C:
）

於「命令提示字元」視窗命令提示列輸入及執行
ASPNET_REGIIS -i