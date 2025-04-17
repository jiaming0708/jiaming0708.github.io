---
title: '[DotnetCore] HttpClient 加上 Header 的作法'
date: 2024-11-27 09:12:04
updated: 2024-11-27 09:12:04
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
thumbnail: /images/dotnet-core.png
---

使用 HttpClient 來呼叫其他的服務取得資料，要加上 header 有幾種作法，底下來看看怎麼用

<!-- more -->

> 環境: Dotnet 9

## 直接加

最直接的作法是在每個 request 前加上 header

```c#
builder.Services.AddHttpClient();

app.MapGet("/weatherforecast", (HttpRequest request, [FromServices] HttpClient httpClient) =>
    {
        httpClient.DefaultRequestHeaders.Add("custom-data", "jimmy test");
        return httpClient.GetFromJsonAsync<IEnumerable<WeatherForecast>>(
            $"http://localhost:5183/weatherforecast");
    })
    .WithName("GetWeatherForecast")
    .WithOpenApi();
```

## 使用 DelegatingHandler

上面的作法很簡單，但每個都要自己加，變得很難以管理
c# 有提供 [DelegatingHandler](https://learn.microsoft.com/en-us/aspnet/web-api/overview/advanced/httpclient-message-handlers) 的做法，這等於是 interceptor 中介，也可以說是 middleware，讓我們可以統一的加上 header 的行為

建立一個 `CustomHttpInterceptor.cs`
```c#
public class CustomHttpInterceptor : DelegatingHandler
{
    private readonly IHttpContextAccessor _httpContextAccessor;

    public CustomHttpInterceptor(IHttpContextAccessor httpContextAccessor)
    {
        _httpContextAccessor = httpContextAccessor;
    }

    protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
        request.Headers.Add("custom-data", "jimmy test");
        // 若有需要取得這次 request 進來的一些資料，可以透過 _httpContextAccessor 取得
        // _httpContextAccessor.HttpContext.Request.Headers.TryGetValue("X-Correlation-Id", out var correlationId);

        // 呼叫下一層處理邏輯
        return await base.SendAsync(request, cancellationToken);
    }
}
```

回到 `Program.cs` 需要調整 `AddHttpClient` 的設定
```c#
builder.Services.AddHttpContextAccessor();
builder.Services.AddTransient<CustomHttpInterceptor>();
// 這邊可以給一個名稱也可以空白
builder.Services.AddHttpClient("test")
    .AddHttpMessageHandler<CustomHttpInterceptor>();
```

上面這些做完原本的注入可以不用改，持續用 `HttpClient`，也可以改用 `IHttpClientFactory`
```c#
app.MapGet("/weatherforecast", (HttpRequest request, [FromServices] IHttpClientFactory httpClientFactory) =>
    {
        var httpClient = httpClientFactory.CreateClient("test");
        return httpClient.GetFromJsonAsync<IEnumerable<WeatherForecast>>(
            $"http://localhost:5183/weatherforecast");
    })
    .WithName("GetWeatherForecast")
    .WithOpenApi();
```

這樣呼叫出去的 API 就都會統一加上 `custom-data` 在 header 上啦!

## Reference
- [Intercepting HTTP requests with a DelegatingHandler](https://timdeschryver.dev/blog/intercepting-http-requests-with-a-delegatinghandler)
- [HttpClient Message Handlers in ASP.NET Web API](https://learn.microsoft.com/en-us/aspnet/web-api/overview/advanced/httpclient-message-handlers)