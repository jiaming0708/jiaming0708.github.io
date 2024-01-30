---
title: '[Blazor] 在 API 伺服器上託管 WASM 遇到的 404 問題'
date: 2024-01-30 13:24:30
updated: 2024-01-30 13:24:30
categories:
- Backend
- C#
tags:
- Blazor
- C#
thumbnail: /images/blazor.jp
---

在 Dotnet 8 以前，微軟有提供過 WASM 可以直接給 API server 託管的功能，等於是只要架一個網站就可以前後端分離，但這樣的設定會衍生打不存在的 API 不會回傳 404，反而是回傳前端 index.html。

<!-- more -->

## 託管設定

想要託管在 API Server，先從 nuget 安裝 `Microsoft.AspNetCore.Components.WebAssembly.Server`，並且在 `Program.cs` 加上兩行，再把前端的專案加入參考。

```c#
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

// 加上這行
app.UseBlazorFrameworkFiles();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapControllers();
// 加上這行
app.MapFallbackToFile("index.html");

app.Run();
```

改完以後，就可以用 server 跑起來，在同一個網站有前後端。

這時候可以測試一個 GET `api/test`，正常預期是 404，但實際上是回傳 `index.html` 的內容。

## 調整

這個問題主要是因為 `MapFallbackToFile` 沒有區分前後端，因此要分開這兩個行為，可以透過 `MapWhen` 來做判斷。

```c#
app.MapWhen(ctx => !ctx.Request.Path.StartsWithSegments("/api"), blazor =>
{
    blazor.UseBlazorFrameworkFiles();
    blazor.UseStaticFiles();
    blazor.UseRouting();

    blazor.UseEndpoints(endpoints => endpoints.MapFallbackToFile("index.html"));
});

app.MapWhen(ctx => ctx.Request.Path.StartsWithSegments("/api"), api =>
{
    api.UseStaticFiles();
    api.UseRouting();
    api.UseAuthentication();

    api.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
        endpoints.MapControllers();
    });
});
```

> 其中 `UseRouting` 必須要在兩邊都寫不能寫在外面，不然可能不會回 404。

這樣改完以後，再次嘗試 `api/test` 就可以得到 404。

## Reference

[c# - How to map fallback in ASP .NET Core Web API so that Blazor WASM app only intercepts requests that are not to the API - Stack Overflow](https://stackoverflow.com/questions/61268117/how-to-map-fallback-in-asp-net-core-web-api-so-that-blazor-wasm-app-only-interc)