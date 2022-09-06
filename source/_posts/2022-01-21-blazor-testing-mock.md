---
title: '[Blazor] 元件測試 - mock'
date: 2022-01-21 15:40:56
categories:
- Frontend
- Blazor
tags:
- Blazor
- Testing
---

前一篇 {% post_link blazor-testing-basic %} 中已經介紹過基本的測試，今天要來介紹的是 `Mock` 的應用。

> 使用上次的 [repo](https://github.com/jiaming0708/blazor-testing-demo) 接續操作

<!-- more -->

# FetchData 頁面

開啟 `Pages` 中的 `FetchData.razor` 這個元件，可以注意到 `OnInitializedAsync` 的部份，是透過 `HttpClient` 打 API 取得資料回來呈現。

```html
@page "/fetchdata"
@inject HttpClient Http

<PageTitle>Weather forecast</PageTitle>

<h1>Weather forecast</h1>

<p>This component demonstrates fetching data from the server.</p>

@if (forecasts == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <table class="table">
        <thead>
            <tr>
                <th>Date</th>
                <th>Temp. (C)</th>
                <th>Temp. (F)</th>
                <th>Summary</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var forecast in forecasts)
            {
                <tr>
                    <td>@forecast.Date.ToShortDateString()</td>
                    <td>@forecast.TemperatureC</td>
                    <td>@forecast.TemperatureF</td>
                    <td>@forecast.Summary</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {
    private WeatherForecast[]? forecasts;

    protected override async Task OnInitializedAsync()
    {
        forecasts = await Http.GetFromJsonAsync<WeatherForecast[]>("sample-data/weather.json");
    }

    public class WeatherForecast
    {
        public DateTime Date { get; set; }

        public int TemperatureC { get; set; }

        public string? Summary { get; set; }

        public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
    }
}
```



## Mock HttpClient

在測試專案中，新增一個檔案 `FetchDataTest.cs`，並加上一個測試案例直接針對 FetchData 這個元件的輸出來看看結果。

```csharp
[Test]
public void RenderWithoutResponse()
{
  using var ctx = new Bunit.TestContext();

  var comp = ctx.RenderComponent<FetchData>();
  Console.WriteLine(comp.Markup);
}
```

應該會看到測試失敗，原因就是在 init 時有使用到 HttpClient。

> This test requires a HttpClient to be supplied, because the component under test invokes the HttpClient during the test. The request that was sent is contained within the 'Request' attribute of this exception. Guidance on mocking the HttpClient is available on bUnit's website.

要對 HttpClient 進行 mock 的話，還需要安裝另外一個 library **RichardSzalay.MockHttp**，打開 Nuget 管理並且加入這個套件到測試專案中。

```powershell
PM> Install-Package RichardSzalay.MockHttp
```

建立一個檔案 `MockHttpClientBunitHelpers`  來實作兩個功能
- 註冊 HttpClient 到 Service
- Response 的介面

```csharp
using System;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text.Json;
using Bunit;
using Microsoft.Extensions.DependencyInjection;
using RichardSzalay.MockHttp;

namespace test;

public static class MockHttpClientBunitHelpers
{
  public static MockHttpMessageHandler AddMockHttpClient(this TestServiceProvider services)
  {
    var mockHttpHandler = new MockHttpMessageHandler();
    var httpClient = mockHttpHandler.ToHttpClient();
    httpClient.BaseAddress = new Uri("http://localhost");
    services.AddSingleton<HttpClient>(httpClient);
    return mockHttpHandler;
  }

  public static MockedRequest RespondJson<T>(this MockedRequest request, T content)
  {
    request.Respond(req =>
    {
      var response = new HttpResponseMessage(HttpStatusCode.OK);
      response.Content = new StringContent(JsonSerializer.Serialize(content));
      response.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
      return response;
    });
    return request;
  }

  public static MockedRequest RespondJson<T>(this MockedRequest request, Func<T> contentProvider)
  {
    request.Respond(req =>
    {
      var response = new HttpResponseMessage(HttpStatusCode.OK);
      response.Content = new StringContent(JsonSerializer.Serialize(contentProvider()));
      response.Content.Headers.ContentType = new MediaTypeHeaderValue("application/json");
      return response;
    });
    return request;
  }
}
```

接著來修改剛剛的測試案例，在 Service 註冊 HttpClient 就可以讓測試順利通過，並且看到結果包含著 **Loading...** 的字串。

```csharp
[Test]
public void RenderWithoutResponse()
{
  using var ctx = new Bunit.TestContext();
  var mock = ctx.Services.AddMockHttpClient();

  var comp = ctx.RenderComponent<FetchData>();
  StringAssert.Contains("Loading...", comp.Markup);
}
```

## mock API

使用 mock 物件提供的 When 以及剛剛所寫的 RespondJson 方法，來處理資料。

```csharp
[Test]
public void RenderWithoutResponse()
{
  using var ctx = new Bunit.TestContext();
  var mock = ctx.Services.AddMockHttpClient();
  mock.When("/sample-data/weather.json").RespondJson(new List<FetchData.WeatherForecast>
  {
    new() {Date = new DateTime(2022, 01, 20), TemperatureC = 15, Summary = "first data"}
  });

  var comp = ctx.RenderComponent<FetchData>();
  StringAssert.Contains("Loading...", comp.Markup);
}
```

可以發現這樣怎麼還是沒重新 render 拿到新的 html 結構，因為是使用非同步的方法取得資料，因此測試也需要使用非同步的方式。

```csharp
[Test]
public void RenderMockResponse()
{
  using var ctx = new Bunit.TestContext();
  var mock = ctx.Services.AddMockHttpClient();
  mock.When("/sample-data/weather.json").RespondJson(new List<FetchData.WeatherForecast>
  {
    new() {Date = new DateTime(2022, 01, 20), TemperatureC = 15, Summary = "first data"}
  });

  var comp = ctx.RenderComponent<FetchData>();
  comp.WaitForAssertion(() =>
  {
    Assert.IsNotNull(comp.Find(".table"));
  });
}
```

除了使用 `WaitForAssertion` 也可以採用另一種等待狀態變更 `WaitForState` 的作法，後面就能變回同步的狀態。

```csharp
  [Test]
  public void RenderMockResponse_WaitState()
  {
    using var ctx = new Bunit.TestContext();
    var mock = ctx.Services.AddMockHttpClient();
    mock.When("/sample-data/weather.json").RespondJson(new List<FetchData.WeatherForecast>
    {
      new() {Date = new DateTime(2022, 01, 20), TemperatureC = 15, Summary = "first data"}
    });

    var comp = ctx.RenderComponent<FetchData>();
    comp.WaitForState(() => !comp.Markup.Contains("Loading..."));
    Assert.IsNotNull(comp.Find(".table"));
  }
```

# Index 頁面

看完 Mock HttpClient 的作法後，接著要來做 Mock Component，有時候我們可以假設 Child component 的行為都是正確的，把測試主力放在自己身上，這時候就會需要這個技巧。

開啟 Pages 中 `Index.razor` 的內容，裡面使用到 `SurveyPrompt` 元件。

```html
@page "/"

<PageTitle>Index</PageTitle>

<h1>Hello, world!</h1>

Welcome to your new app.

<SurveyPrompt Title="How is Blazor working for you?" />
```

## Mock Component

在測試專案中，新增檔案 `IndexTest.cs`。
在 ComponntFactories 加上要 mock 的元件，先試著 mock SurveyPrompt 看輸出內容，就可以發現那個元件的部份是完全空白。

```csharp
[Test]
public void MockChildComp()
{
  using var ctx = new Bunit.TestContext();
  ctx.ComponentFactories.AddStub<SurveyPrompt>();

  var comp = ctx.RenderComponent<Index>();
  Console.WriteLine(comp.Markup);
  /*
<h1>Hello, world!</h1>

Welcome to your new app.
  */
}
```

可以使用 `IRenderedComponent.HasComponent<T>()` 檢查元件是否有存在，將測試案例調整一下。

```csharp
[Test]
public void MockChildComp()
{
  using var ctx = new Bunit.TestContext();
  ctx.ComponentFactories.AddStub<SurveyPrompt>();

  var comp = ctx.RenderComponent<Index>();
  Assert.False(comp.HasComponent<SurveyPrompt>());
  Assert.True(comp.HasComponent<Stub<SurveyPrompt>>());
}
```

## Mock content

剛剛的測試是沒有任何內容，如果想要自訂內容的話，可以在 `AddStub<T>("xxxx")` 中放入字串。

```csharp
[Test]
public void MockChildCompContent()
{
  using var ctx = new Bunit.TestContext();
  var content = "<div>Mock SurveyPrompt</div>";
  ctx.ComponentFactories.AddStub<SurveyPrompt>(content);

  var comp = ctx.RenderComponent<Index>();
  StringAssert.Contains(content, comp.Markup);
}
```

## Mock content with parameter

除了寫死的內容以外，也能夠支援傳遞參數的方法，在 AddStub 的方法內改用函式的作法，來取得並輸出參數。

```csharp
[Test]
public void MockChildCompContentWithParameter()
{
  using var ctx = new Bunit.TestContext();
  ctx.ComponentFactories.AddStub<SurveyPrompt>(paras => $"<div>{paras.Get(x => x.Title)}</div>");

  var comp = ctx.RenderComponent<Index>();
  StringAssert.Contains("How is Blazor working for you?", comp.Markup);
}
```

## 使用第三方套件

除了 bUnit 的這種作法外，也可以使用第三方的 mock 套件，像是 `MOQ` 及 `NSubtitute`。作法跟上面的類似，將 `AddStub<T>()` 改為 `Add<T>(mock object)` 。

打開 Nuget 管理將 `MOQ` 這個套件加入到測試專案中，這邊將使用 MOQ 做範例。

```powershell
PM> Install-Package MOQ
```

```csharp
[Test]
public void MockComponentByMOQ()
{
  using var ctx = new Bunit.TestContext();
  var mockComp = new Mock<SurveyPrompt>();
  ctx.ComponentFactories.Add(mockComp.Object);
  // 另一種寫法
  // ctx.ComponentFactories.Add(() => Mock.Of<SurveyPrompt>());

  var comp = ctx.RenderComponent<Index>();
  var actualComp = comp.FindComponent<SurveyPrompt>();
  Assert.AreSame(mockComp.Object, actualComp.Instance);
}
```

# 總結

今天介紹了 HttpClient 及 Component 的 mock 方法，個人覺得還要另外採用第三方套件來做 Mock 是蠻麻煩的，但官方目前的態度就是這樣，bUnit 不去依賴第三方套件。

[範例程式](https://github.com/jiaming0708/blazor-testing-demo) 也放在 github 上，如果有需要也可以按照 commit 的步驟來跟著操作學習。

# Reference

* [mocking component](https://bunit.dev/docs/providing-input/substituting-components.html?tabs=moq)
* [mocking HttpClient](https://bunit.dev/docs/test-doubles/mocking-httpclient.html)
