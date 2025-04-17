---
title: '[DotnetCore] Options 模式的使用方法'
date: 2025-04-17 08:44:29
updated: 2025-04-17 08:44:29
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
thumbnail: /images/dotnet-core.png
---

最近看到同事使用 Options 模式，不確定是否有更好的實現方式，因此決定重新複習 Options 在 ASP.NET 中的使用方法。

<!-- more -->

首先，在 `appsettings.json` 中定義一組設定，並透過依賴注入（DI）機制將其轉換為物件來使用。

``` json
  "test": {
    "key": "testValue",
    "value": 1
  }
```

```c#
public class TestOptions
{
    public string Key { get; set; }
    public int Value { get; set; }
}

// Program.cs
builder.Services.AddOptions<TestOptions>().Bind(builder.Configuration.GetSection("Test"));

app.MapGet("/weatherforecast", (IOptions<TestOptions> options) =>
{
  Console.WriteLine(options.Value.Key);
}
```

使用 `AddOptions` 方法將 Configuration 中的 `Test` 設定讀取進來，並透過 DI 使用 `IOptions<T>` 來取得該設定資料。

接著，我們需要在 `appsettings.json` 中新增另一組設定。

```json
  "test": {
    "key": "testValue",
    "value": 1
  },
  "test2": {
    "key": "testValue2",
    "value": 2
  }
```

同事的做法是定義一個新的類別 `Test2Options` 來讀取新的設定。這樣看起來沒問題，但既然兩個結構完全相同，為什麼不能共用同一個類別呢？

如果遇到類似需求，可以在 `AddOptions` 時指定名稱，並透過 DI 根據名稱取得對應的設定資料。以下是範例：

```c#
// Program.cs
builder.Services.AddOptions<TestOptions>("test").Bind(builder.Configuration.GetSection("Test"));
builder.Services.AddOptions<TestOptions>("test2").Bind(builder.Configuration.GetSection("Test2"));

app.MapGet("/weatherforecast", (IOptionsSnapshot<TestOptions> options) =>
{
  var testOptions = options.Get("test");
  var test2Options = options.Get("test2");
}

```

原本的 `IOptions<T>` 不支援根據名稱取得不同設定，因此需要使用 `IOptionsSnapshot<T>` 或 `IOptionsMonitor<T>` 來實現。

`IOptionsSnapshot` 是 Scoped，適合用於在 app 啟動後資料可能變更的情境，並能取得最新資料；而 `IOptionsMonitor` 的主要差異在於它是 Singleton。
> 參考 [Learn 文件](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-9.0#use-ioptionssnapshot-to-read-updated-data)

以上就是對 Options 模式的簡單複習。
