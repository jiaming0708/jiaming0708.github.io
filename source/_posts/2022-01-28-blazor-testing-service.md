---
title: '[Blazor] 元件測試 - Service'
date: 2022-01-28 09:25:36
categories:
- Frontend
- Blazor
tags:
- Blazor
- Testing
---

這是系列文的第三篇，今天要講到的是針對 Service 做測試。
如果還沒看過前面兩篇的話，請往這邊走 {% post_link blazor-testing-basic %}、{% post_link blazor-testing-mock %}。

> 使用上次的 [repo](https://github.com/jiaming0708/blazor-testing-demo) 接續操作

<!-- more -->

# 重構成 service

在原本的 FetchData.razor 頁面中，是直接用 HttpClient 呼叫 API，今天的測試重點是 Service ，因此需要先將使用 HttpClient 的部分抽出成 Service。
建立資料夾 `Services` 及檔案 `IDataService.cs` 。

```csharp
using System.Threading.Tasks;
using client.Pages;

namespace client.Services
{
    public interface IDataService
    {
        public Task<FetchData.WeatherForecast[]?> GetWeatherForecast();
    }
}
```

再建立一個檔案 `DataService.cs`  來繼承 `IDataService`，並且使用透過 DI 的方法取得 HttpClient。

```csharp
using System.Net.Http;
using System.Net.Http.Json;
using System.Threading.Tasks;
using client.Pages;

namespace client.Services
{
    public class DataService: IDataService
    {
        private HttpClient _http;

        public DataService(HttpClient http)
        {
            _http = http;
        }

        public async Task<FetchData.WeatherForecast[]?> GetWeatherForecast()
        {
            return await _http.GetFromJsonAsync<FetchData.WeatherForecast[]>("sample-data/weather.json");
        }
    }
}
```

Blazor 要用到 Service 的話需要在 Program.cs 中宣告。

```csharp
var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

builder.Services.AddScoped<IDataService, DataService>(); // <== 定義 DataService 及 IDataService 的關係
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });

await builder.Build().RunAsync();
```

最後一個步驟，就是將 FetchData.razor 改呼叫 Service。

```html
@page "/fetchdata"
@using client.Services
@inject IDataService _dataService;

<!-- html 省略 -->

@code {
    private WeatherForecast[]? forecasts;

    protected override async Task OnInitializedAsync()
    {
        forecasts = await _dataService.GetWeatherForecast();
    }

	//...
}
```

# 修改測試

都改完以後先執行看看，確認功能跑起來是否正確，並且檢查測試案例是否通過，不出意料測試案例果然是失敗的，因為多了一個 Service。
Service 在 bUnit 中的做法跟 blazor 差不多，都是要加入 `Services` 的集合中。

```csharp
[Test]
public void RenderWithoutResponse()
{
using var ctx = new Bunit.TestContext();
ctx.Services.AddScoped<IDataService, DataService>(); // <=== 定義 DataService 及 IDataService 的關係
var mock = ctx.Services.AddMockHttpClient();

var comp = ctx.RenderComponent<FetchData>();
StringAssert.Contains("Loading...", comp.Markup);
}
```

# Mock Service

DataService 是繼承 Interface，有兩種做法可以使用

* 建立另一個的 DataService 給測試使用
* 使用 `MOQ` / `NSubtitute`

在前一篇已經安裝過 MOQ ，這邊就繼續採用他來作為測試的方法。

```csharp
[Test]
public void MockService()
{
using var ctx = new Bunit.TestContext();

var mockService = new Mock<IDataService>();
mockService
  .Setup(p => p.GetWeatherForecast())
  .ReturnsAsync(new List<FetchData.WeatherForecast>
  {
    new() {Date = new DateTime(2022, 01, 20), TemperatureC = 15, Summary = "first data"}
  }.ToArray());
ctx.Services.AddSingleton<IDataService>(mockService.Object);

var comp = ctx.RenderComponent<FetchData>();
comp.WaitForState(() => !comp.Markup.Contains("Loading..."));
Assert.IsNotNull(comp.Find(".table"));
}
```

> 透過 Mock Interface 的方法搭配上使用 AddSignleton。

# Reference

- [Quickstart · moq/moq4 Wiki (github.com)](https://github.com/Moq/moq4/wiki/Quickstart#async-methods)
- [Awaiting an asynchronous state change in a component under test | bUnit](https://bunit.dev/docs/interaction/awaiting-async-state.html)
