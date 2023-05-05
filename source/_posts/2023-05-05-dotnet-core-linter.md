---
title: '[DotnetCore] 啟用 Linter 做把關'
date: 2023-05-05 17:32:46
updated: 2023-05-05 17:32:46
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
thumbnail:
---

在 JS 的世界中，有 [ESLinter](https://eslint.org/) 可以幫助我們做一些排版還有變數等等的檢查，提升程式碼風格一致性，在 Dotnet Core 也希望有類似的功能。

<!-- more -->

我知道的作法有以下幾個:

- 透過 `SonarQube` 在 CI 環節中檢查，推上去被退件還要再重新推才行，算是事後的檢查跟 ESLinter 的角色有點不同。
- 使用 cli 的指令 `dotnet format`，可以寫在 git hook 或是 build event 整合在一起。
- 使用 [Roslyn Analyzers](https://github.com/dotnet/roslyn-analyzers) 在編譯的時候檢查，VS 或 Rider 都會認識並且在撰寫的階段就提示。

## 使用 format

在 dotnet core 專案的目錄下開啟 Terminal，執行指令

```shell
dotnet new editorconfig
```

這個指令會產生 `.editorconfig` 的檔案，裡面定義了各個規則，設定內縮用空格還是 Tab，宣告變數時使用 var 還是實際型別等等。
當什麼都沒改的情況下，將下面這段程式的格式弄亂一點

```c#
[HttpGet(Name = "GetWeatherForecast")]
public IEnumerable<WeatherForecast> Get()
{
       return Enumerable.Range(1,5) .Select( index=>new WeatherForecast{
        Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
        TemperatureC = Random.Shared.Next(-20, 55),
        Summary = Summaries[Random.Shared.Next(Summaries.Length)]
    }).ToArray();
}
```

一樣在 terminal 執行指令，要求進行程式碼格式化

```c#
dotnet format
```

如下所示可以看到排版後的結果，原本一些沒有空格或是多空格的地方，都幫你排好了

```c#
[HttpGet(Name = "GetWeatherForecast")]
public IEnumerable<WeatherForecast> Get()
{
    return Enumerable.Range(1, 5).Select(index => new WeatherForecast
    {
        Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
        TemperatureC = Random.Shared.Next(-20, 55),
        Summary = Summaries[Random.Shared.Next(Summaries.Length)]
    }).ToArray();
}
```

## 編譯強制執行

前面的 format 有達到預期的一小部分，但更希望的是在 commit 或是編譯時就提醒，接下來就進一步的要求 dotnet 在編譯的時候就有錯誤失敗。

開啟專案的 `csproj` ，找到 `PropertyGroup` 的區塊，在裡面加上 `EnforceCodeStyleInBuild` 這個標籤，並設定為 true。

```xml
<PropertyGroup>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <!-- 在編譯時檢查 coding style -->
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
</PropertyGroup>
```

開啟剛剛產生的 `.editorconfig` 檔案，找到 `[*.cs]` 區塊底下加上底下的設定，直接開啟檢查等級為 `error`，也就是說只要寫法跟設定內的規則不同就會當作錯誤。

```ini
# C# files
[*.cs]
# Default severity for analyzer diagnostics with category 'Style' (escalated to build errors)
dotnet_analyzer_diagnostic.category-Style.severity = error
```

並且把前面那段格式有問題的程式貼回去，還不用編譯就會在 VS 看到一堆紅蚯蚓

![vs-linter-error](vs-linter-error.png)

不知道怎麼修的話，可以用 VS 的建議來直接修復

![vs-linter-error-fix](vs-linter-error-fix.png)

## 結論

開啟 Linter 的好處就是很多寫法是有規範的，可以避免掉一些很基礎的問題讓自己不爽，但預設的規則也很硬，要開啟的話記得要跟團隊好好討論一下要怎麼弄，不然絕對會搞到自己。

> 像是 using 要分門別類的排好以外還要空行

另外就是有一些很新的規則，還沒有文件可以查，但在 linux 的 sdk 7.0 image 中會預設啟用，就比較麻煩要去查 github 才會有，例如 [IDE0270](https://github.com/dotnet/roslyn/issues/65769)。


## Reference

- [ESLint your C# in VS Code with Roslyn Analyzers | johnnyreilly](https://johnnyreilly.com/eslint-your-csharp-in-vs-code-with-roslyn-analyzers)
- [Code analysis in .NET | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/overview?tabs=net-7)
