---
title: '[Blazor] 元件測試 - 基礎篇'
date: 2021-12-23 22:27:50
categories:
- Front-end
- Blazor
tags:
- Blazor
- Testing
---

寫了很多的元件來使用，但要怎麼確保行為正確，在修改的時候沒有把一些行為改壞，可以透過`測試`來進行保護，在 Blazor 中可以透過 [**bUnit**](https://bunit.dev/) 這個套件來達成。

<!-- more -->

bUnit 可以套用在 xUnit/NUnit/MSTest 任何一個測試框架上，寫法不會因為框架的不同而有所差異。跟 API 或是 **method** 不一樣的測試目的，元件通常是由一個 click 或是外部的參數改變，而產生一些行為的變化，在元件的測試中，

# 環境建置

在開始之前要先準備一下專案的環境，直接用 cli 來產生 blazor wasm 的專案，或者用 VS/Rider 這種 IDE 來產生

> dotnet sdk 版本為 6

```shell
mkdir blazor-tsting-demo
dotnet new sln -n blazor-testing-demo
dotnet new blazorwasm -n client
dotnet sln add ./client/client.csproj
```

接著在把測試專案也先建立好，並且增加 bunit 及 client 的參考到專案中。

> 我使用的是 **nunit** ，如果你有習慣的測試框架也可以使用沒有問題

```shell
dotnet new nunit -n test
dotnet sln add ./test/test.csproj
cd test
dotnet add package bunit
dotnet add test.csproj reference ../client/client.csproj
```

# 撰寫測試

## Render驗證

不管透過 cli/IDE 所建立出來的 blazor wasm 專案都有基本的樣板，這邊就先來看一下樣板中 `Pages/Counter.razor` 這個元件，有什麼樣的行為，並且如何進行測試。

```html
@page "/counter"

<PageTitle>Counter</PageTitle>

<h1>Counter</h1>

<p role="status">Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

在這個樣板中，主要的行為是按下 button 後 count 會加1，測試的動作，就是要模擬使用者點擊 button 後，html 應該要反映出 count 的變化。

回到測試專案中，把原本的 `UnitTest1.cs` 改成 `CounterTest.cs` ，之後我們還會繼續增加其他的測試檔案。
將原本的 `Test1` 改成 `CounterRender` ，宣告一個 Bunit 的 **TestContext** 並且呼叫 **RenderComponent** 就可以取得 Counter 這個元件的 html 輸出。

```csharp
[Test]
public void CounterRender()
{
  using var ctx = new Bunit.TestContext();

  var comp = ctx.RenderComponent<Counter>();
  Console.WriteLine(comp.Markup);
/*
<h1>Counter</h1>

<p role="status">Current count: 0</p>

<button class="btn btn-primary" blazor:onclick="1">Click me</button>
*/
}
```

這樣的輸出結果，會發現兩個比較困難的地方

* 換行字元
* `blazer:onclick` 屬性並不知道內容是什麼

我會建議採用部分比對就可以，抓到重點確定內容有符合即可，將測試案例調整如下

```csharp
[Test]
public void CounterRender()
{
  using var ctx = new Bunit.TestContext();

  var comp = ctx.RenderComponent<Counter>();
  StringAssert.Contains("Current count: 0", comp.Markup);
}
```

![CounterRender test pass](./counter-render-pass.png)

## 互動驗證

第一個測試案例寫好後，接著要來寫第二個案例，點擊 button 後 count 應該加1。
從 render 出來的元件中找到需要的 element，並且進行互動。

```csharp
[Test]
public void CounterShouldIncrementWhenClicked()
{
  using var ctx = new Bunit.TestContext();

  var comp = ctx.RenderComponent<Counter>();
  comp.Find("button").Click();

  StringAssert.Contains("Current count: 1", comp.Markup);
}
```

或者是直接拿到 p 這個 element，來驗證裡面的內容

```csharp
comp.Find("p").TextContent.MarkupMatches("Current count: 1");
```

![counter-increment-clicked](./counter-increment-clicked.png)

## 比對驗證

畫面有了異動，會想要知道異動的地方是不是如我們所預期，這時候可以透過比對差異的方法，有兩種作法可以做到

* 跟第一次比

```csharp
[Test]
public void CounterShouldIncrementWhenClicked_CompareWithFirst()
{
  using var ctx = new Bunit.TestContext();

  var comp = ctx.RenderComponent<Counter>();
  comp.Find("button").Click();

  var diffs = comp.GetChangesSinceFirstRender();
  var diff = diffs.ShouldHaveSingleChange();
  diff.ShouldBeTextChange("Current count: 1");
}
```

![compare-first](./compare-first.png)

* 比對 `snapshot`，在比對之前要先儲存

  > 在這邊的範例中比較展示不出來差異，因為異動的都是同一個地方。

```csharp
[Test]
public void CounterShouldIncrementWhenClicked_CompareWithSnapshot()
{
  using var ctx = new Bunit.TestContext();

  var comp = ctx.RenderComponent<Counter>();
  var button = comp.Find("button");
  button.Click();

  comp.SaveSnapshot();

  button.Click();

  var diffs = comp.GetChangesSinceSnapshot();
  var diff = diffs.ShouldHaveSingleChange();
  diff.ShouldBeTextChange("Current count: 2");
}
```

![compare-shapshot](./compare-shapshot.png)

## 參數給值

接著要來改一下功能需求，可以由外面來決定初始 Count 是多少，沒有給的話就是0。

````razor
@code {
  [Parameter]
  public int currentCount { get; set; } = 0;

  private void IncrementCount()
  {
    currentCount++;
  }
}
````

將剛剛所寫的幾個測試案例重跑過一遍，要確定這樣的修改是正確的。

![image-20211223212905835](./counter-testing-after-change.png)

測試通過後，再來寫傳遞參數的測試案例，有兩個方法可以傳遞參數

* 在 init 時就給

```csharp
[Test]
public void CountShouldBe5_OnInit()
{
  using var ctx = new Bunit.TestContext();

  var count = 5;
  var comp = ctx.RenderComponent<Counter>(parameters => parameters.Add(p => p.currentCount, count));
  comp.Find("p").TextContent.MarkupMatches($"Current count: {count}");
}
```

![parameter-on-init](./parameter-on-init.png)

* 先 init 再給值

```csharp
[Test]
public void CountShouldBe5_AfterInit()
{
  using var ctx = new Bunit.TestContext();

  var count = 5;
  var comp = ctx.RenderComponent<Counter>();
  var elm = comp.Find("p");
  elm.TextContent.MarkupMatches("Current count: 0");

  comp.SetParametersAndRender(parameters => parameters.Add(p => p.currentCount, count));
  elm.TextContent.MarkupMatches($"Current count: {count}");
}
```

![parameter-after-init](./parameter-after-init.png)

# 總結

今天的重點會放在元件的基本互動，click 還有 parameter 進行測試，可以看到其實都是以 html 的變化作為測試結果的確認，也很符合元件給使用者操作的印象。

今天的 [範例程式](https://github.com/jiaming0708/blazor-testing-demo) 也放在 github 上，如果有需要也可以按照 commit 的步驟來跟著操作學習。

# Reference

* [verify markup](https://bunit.dev/docs/verification/verify-markup.html)
* [parameter](https://bunit.dev/docs/providing-input/passing-parameters-to-components.html?tabs=csharp)
