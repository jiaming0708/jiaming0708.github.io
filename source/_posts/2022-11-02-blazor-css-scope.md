---
title: '[Blazor] 搞懂 css scope'
date: 2022-11-02 20:42:21
categories:
- Frontend
- Blazor
tags:
- Blazor
thumbnail: /images/blazor.jpg
---

Blazor 的 css 作用域預設是元件內有效，那如果想要往子元件渲染的話，可以怎麼做？今天要來搞懂 Blazor 的 css scope 是怎麼樣運作，let's go!

<!-- more -->

> 環境:
> dotnet 6
> blazor wasm

使用 dotnet cli 來產生一個 blazor web assembly 的專案，應該可以看到以下的內容

```shell
dotnet new blazorwasm -n blazor-css-scope
```

![project-files](project-files.png)

什麼都不動的情況下，有看到兩個元件有自己的 css file，一個是 `MainLayout` 另一個則是 `NavMenu`，這兩個都是基底的元件，任一個畫面都會使用到。
我們就直接跑起來看看網站的 f12 是長什麼樣

```shell
dotnet watch run
```

## 預設的行為

這邊是 MainLayout.razor 內容，可以看到有 `page`, `sidebar` 這些的 class 宣告

![MainLayout](MainLayout.png)

接著打開網站開發者工具，檢視元素的部份，找到 `id="app"` 並且打開裡面的節點，可以看到 MainLayout 這個元件，有 `page`, `sidebar` 這些的 class 宣告。

> 不知道你有沒有注意到一個小地方，每個元素上面都被加上了 `b-jrym99nura` 這個屬性，等等會用到喔！

![MainLayoutHtml](MainLayoutHtml.png)

點到 page 的元素，看到 style 的渲染結果，原本的 `page` class 還被添加了一個奇怪的屬性。

![page-style](page-style.png)

```css
.page[b-jrym99nura] {
  position: relative;
  display: flex;
}
```

blazor 為了避免各個元件的 css 互相的干擾，加上了這個機制，在該元件的每個元素上都給一個唯一的編碼作為屬性，對應的 css 也會加上這個屬性。
也就是說，不同的元件，就會有不同的編碼，子元件也不會這樣就被影響到樣式。

> 前端三御家，其實也有差不多的作法，用來控制樣式不會互相污染

這邊我們可以用 MainLayout 裡面的 `NavMenu` 這個元件來看，可以發現到跟 MainLayout 的編碼不同。

![different-id](different-id.png)

## 跨元件渲染

從前面例子可以看到 blazor 預設是不會去影響子元件，想要跨元件的話，blazor 也有提供一個關鍵字 `::deep`，可以讓編譯器知道作用域是要往下影響。

寫一個元件 `CssScope.razor` 裡面放兩個 h1，來做為測試用。

```html
<h1>hi css</h1>
<h1>hi scope</h1>
```

在 `Index.razor` 加上 `CssScope` 的元件。

```html
<h1>Hello, world!</h1>

Welcome to your new app.

<CssScope />

<SurveyPrompt Title="How is Blazor working for you?" />
```

在 `Index.razor.css` 中去改變 h1 的顏色為紅色。

```css
h1 {
	color: red;
}
```

這樣寫就如同前面所提到的只會渲染到自己的元件，底下的元件是不會理，那我們試著加上 `::deep` 看看。

```css
::deep h1 {
	color: red;
}
```

8bq了，底下元件沒渲染到就算，連自己都沒了，這到底有沒有實際的效果呢？
不要緊張，開啟開發者工具，在 source 中可以找到專案的 css 檔案 `blazor-css-scope.styles.css`，屬於這個專案的樣式全部都會集中在這個檔案中。
我們要從裡面來看看到底 css 被定義成什麼樣的內容，剛好第一段就是我們要找的。

```css
/* /Pages/Index.razor.rz.scp.css */
[b-hrxucffzsm] h1 {
  color: red;
}
```

可以看到被編譯成擁有 `b-hrxucffzsm` 這個屬性底下的所有 h1 都可以套用到這個結果，那就好奇這個編碼有被套用在哪些元素身上。

![index-scope](index-scope.png)

只有 hellow world 的 h1 被加上編碼屬性，那依照剛剛的 css 規則來說，等於沒有任何元素是在那個編碼屬性底下。
要讓 css 能應用到子元件，有幾個作法可以考慮看看

- 使用 `~`，接在 index 編碼屬性後的 h1 都套用

  > 這缺點就是只能是同層，如果 h1 有被其他元素包住，就沒有效果

```css
::deep ~ h1 {
	color: red;
}
```

- 用一個元素來包住子元素，原本的 css 規則不用動

  > 不管子元素的 h1 有幾層，都可以被套用到

```html
<div>
  <h1>Hello, world!</h1>

  Welcome to your new app.

  <CssScope />
</div>
```

這邊的觀念其實是回到了 css 規則的部份，在開發的時候善用開發者工具，可以幫助到偵錯前端的問題。

> 對於規則沒那麼熟悉的話，推薦可以看 Amos 的 [金魚都能懂的 CSS 選取器](https://ithelp.ithome.com.tw/users/20112550/ironman/2799)。

## Reference

- [ASP.NET Core Blazor CSS isolation](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/css-isolation?view=aspnetcore-6.0)
