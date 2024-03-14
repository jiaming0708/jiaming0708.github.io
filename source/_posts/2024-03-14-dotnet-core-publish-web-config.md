---
title: '[DotnetCore] web.config 發布控制'
date: 2024-03-14 09:10:46
updated: 2024-03-14 09:10:46
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
thumbnail: 
---

Dotnet Core 在發布的時候預設會輸出 web.config，發佈到 IIS 伺服器如果有要調整 web.config 的內容，可以用以下幾種做法來達成。

<!-- more -->

## 關閉輸出

在 server 上修改，不要讓重新佈署的時候被覆蓋，可以使用 `IsTransformWebConfigDisabled` 來關閉輸出。

- 使用指令

```shell
dotnet publish -c Release -p:IsTransformWebConfigDisabled=true
```

- 專案檔 csproj 設定

```xml
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
  </PropertyGroup>
```

## 建立 web.config

如果是希望每個發布的 web.config 都有同樣的內容，那也可以建立 web.config 在專案中。
標準沒有什麼設定的情況下，輸出的檔案內容如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location>
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" arguments=".\Server.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

增加一個環境變數在檔案內，可以只設定額外增加的內容，剩下的系統會自動幫你填上

```xml
<configuration>
  <location>
    <system.webServer>
        <aspNetCore>
          <environmentVariables>
            <environmentVariable name="Configuration_Specific" value="Configuration_Specific_Value" />
          </environmentVariables>
        </aspNetCore>
    </system.webServer>
  </location>
</configuration>
```

上面的內容在執行 publish 指令後會是長這樣

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location>
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" arguments=".\Cimes.Server.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess">
        <environmentVariables>
          <environmentVariable name="Configuration_Specific" value="Configuration_Specific_Value" />
        </environmentVariables>
      </aspNetCore>
    </system.webServer>
  </location>
</configuration>
```

## Reference

- [web.config file | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/web-config?view=aspnetcore-8.0)

- [如何透過 dotnet publish 調整 ASP․NET Core 部署到 IIS 的 Web.config 內容 | The Will Will Web (miniasp.com)](https://blog.miniasp.com/post/2023/03/30/How-to-set-environment-name-for-webconfig-when-run-dotnet-publish)