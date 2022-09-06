---
title: 在blazor中使用JavaScript
date: 2021-06-27 10:08:40
categories:
- Frontend
- Blazor
tags:
- Blazor
- JavaScript
---

最近都在使用一個很新的前端框架，叫做 [Blazor](https://docs.microsoft.com/zh-tw/aspnet/core/blazor/?view=aspnetcore-5.0) ，是由微軟所推出給 dotnet 的開發人員所用，優點是可以使用一個語言打天下(迷之音: 真的能嗎?)，但現在的 Blazor 還太年輕，對應的東西都還沒那麼齊全，JavaScript 反而已經擁有很多套件/方法可以使用，今天就一起來看看怎麼將兩個結合一起使用。

<!-- more -->

> **環境**
>dotnet core 5
> blazor webassembly

# 基本使用

首先，先來介紹一點基本的應用，如何在 Blazor 呼叫 JS。先準備好一個 **index.js** 的檔案，並且在 html 中載入。接下來的 js 範例都會放在這個檔案中。

```html
<html>
    <head>
        <script src="./scripts/index.js"></script>
    </head>
</html>
```

## 呼叫方法

Blazor 是一個以 dotnet 為基底的框架，因此沒有辦法直接呼叫 js，要使用 [IJSRuntime](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.jsinterop.ijsruntime?view=dotnet-plat-ext-6.0) 這個介面，裡面提供的 `InvokeAsync` 或是 `InvokeVoidAsync` 兩個方法。
然後有一點要特別注意，不管是哪一個方法都只能呼叫 `function` ，因此要把需要執行的指令包裝過，先來宣告一個 function 叫做 `HelloWorld`。

```javascript
function HelloWorld(){
    console.log('hello world');
}
```

接著在 Blazor 使用點擊按鈕來呼叫js。

```html
@page "/sample"
@inject IJSRuntime JS

<button @onclick="clickConsole"> console </button>

@code {
	async Task clickConsole()
	{
		await JS.InvokeVoidAysnc("HelloWorld");
	}
}
```

## 傳遞參數

通常我們會希望把資料丟給 js 或是從 js 拿到一些回傳值，先來看看怎麼把資料傳給 js。
在 `InvokeVoidAsync` 的第二個參數開始，就是傳給 js function 的所有參數。

```javascript
function HiName(name){
    console.log(`Hi! ${name}`);
}
```

```c#
async Task clickConsole()
{
	await JS.InvokeVoidAysnc("HiName", "jimmy");
}
```

## 接受回傳值

那如果說，我們要拿到 js 的回傳值，就要改用 `InvokeAsync` 這個方法才能接收到。

```javascript
function getElementHeight(id) {
    const elm = document.querySelector(`#${id}`);
    return elm.offsetHeight;
}
```

```c#
async Task clickConsole()
{
	var height = await JS.InvokeAysnc<int>("getElementHeight", "block1");
}
```

# 動態載入

執行到某個頁面的時候才載入一些 js 的檔案，減少第一次不必要的下載，對於前端來說是蠻常見的事情，在 blazor 中當然也有提供這樣的方法。

這邊準備另一個檔案叫做 **module.js** ，然後將前面的 `HelloWorld` 放進去，來測試看看。

## 嘗試

在 blazor 這邊的做法其實一樣，沒有其他的方法可以使用，但參數要稍微改一下，第一個參數使用 `import` ，第二個參數給`路徑`，然後回傳的值就會是整個 module 的物件，。

```c#
async Task clickConsole()
{
	var jsmodule = await JS.InvokeAysnc<IJSObjectReference>("import", "./scripts/module.js");
    jsmodule.InvokeVoidAysnc("HelloWorld");
}
```

## 寫法調整

如果你按照這樣寫，應該會得到錯誤，找不到 `HelloWorld` 這個方法。
原因是使用 module 的作法，整個就會是封閉的，外面如果要調用，就必須要加上 **export**。

> 如果對於 module 的寫法不太熟悉，可以參考一下 MDN 的文件，[JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)

```javascript
export function HelloWorld(){
    console.log('hello world');
}
```

# 應用分享

最後來分享幾個自己在開發上會使用到的技巧/應用。

## 傳入 blazor 自己

在使用 chart 套件的時候，很常會有互動的行為，那就會需要從套件中來呼叫 blazor 的方法，官方有給出幾個做法，這邊我只介紹其中之一，個人覺得最乾淨的。
產生一個該類別的物件，再傳遞給 js 來作為回呼的物件。

```c#
async Task clickDraw()
{
	var objRef = DotNetObjectReference.Create(this);
	var height = await JS.InvokeAysnc<int>("DrawChart", objRef);
}

[JSInvokable]
public void onClick()
{
    // ...
}
```

```javascript
function DrawChart(dotnetHelper){
	new Chart('#chart', {
		on_click: function(){
			dotnetHelper.invokeMethodAsync('onClick');
		}
	});
}
```

## 回傳的物件型態

從 js 的回傳物件中，如果不是基礎類別，也是可以透過類別來轉型，但這樣的物件再次丟回給 js 時，會導致無法正常找到一些 property，這時候可以使用 **IJSObjectReference** 這個型別。

```javascript
function DrawChart(dotnetHelper){
	return new Chart('#chart', {
		on_click: function(){
			dotnetHelper.invokeMethodAsync('onClick');
		}
	});
}
```

```c#
async Task clickDraw()
{
	var objRef = DotNetObjectReference.Create(this);
	var chartObj = await JS.InvokeAysnc<IJSObjectReference>("DrawChart", objRef);
}
```
## 動態載入的動態載入

在檔案載入的時候，希望能夠直接初始化做一些事情，例如引入第三方套件的 js/css，在這邊我們要使用 js 的一個技巧叫做 [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) (立即執行函式)，並且使用 `createElement` 的作法來做到在檔案載入後才去引入對應的 js/css。

```javascript
(function (d, id) {
    // not create script element again if existing
    if (d.getElementById(id)) {
        return;
    }

    const headElm = d.getElementsByTagName("head")[0]

    const linkElm = d.createElement("link");
    linkElm.rel = 'stylesheet';
    linkElm.type = 'text/css';
    linkElm.href = 'js/frappe-gantt/dist/frappe-gantt.css';
    headElm.appendChild(linkElm);

    const scriptElm = d.createElement('script');
    scriptElm.id = id;
    scriptElm.src = 'js/frappe-gantt/dist/frappe-gantt.min.js';
    headElm.appendChild(scriptElm);
}(document, 'gantt-script'));
```

# 參考

* [Blazor](https://docs.microsoft.com/zh-tw/aspnet/core/blazor/?view=aspnetcore-5.0)
* [Call JavaScript functions from .NET methods in ASP.NET Core Blazor | Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/blazor/javascript-interoperability/call-javascript-from-dotnet?view=aspnetcore-5.0)
* [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)
* [JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)