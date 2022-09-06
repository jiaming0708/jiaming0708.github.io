---
title: '[Blazor] 客製化元件'
date: 2021-07-03 09:27:13
categories:
- Frontend
- Blazor
tags:
- Blazor
---

不管是哪一個前端的框架或套件，寫一個專屬於自己專案的元件，是很基本的行為，今天要來分享的是在 Blazor 中建立元件的一些小撇步。

<!-- more -->

# two-way binding (雙向綁定)

## 基本使用

Blazor 是一個標榜可以做到雙向綁定的一個前端框架，先來看看最基本的範例，使用 `bind` 這個前綴就可以做到雙向綁定，等於是把 `value` 以及 `onchange` 做整合。

```razor
<input @bind="data" />
<input value="@data" @onchange="@((ChangeEventArgs __e) => data = __e.Value.ToString())" />

@code {
  string data;
}
```

透過上面的範例，可以發現到這兩個行為是相同的，從這邊可以理解到所謂的雙向綁定，也就是將做到輸入以及資料的輸出兩個行為做結合。

## 元件雙向綁定

根據上面的行為，我們要來嘗試所謂的雙向綁定，在 blazor 中有一個規則，只要事件的名稱結尾是 **Changed** 就可以搭配 bind 做雙向綁定。

底下我們先寫一個元件來測試雙向綁定是否正確，元件的名稱就直接叫做 **Jimmy.razor**，並且公開一個屬性叫做 `Value`。

```razor
<button @onclick="clickChange">change value</button>

@code {
    [Parameter] public string Value { get; set; }
    [Parameter] public EventCallback<string> ValueChanged { get; set; }

    void clickChange()
    {
        ValueChanged.InvokeAsync("jimmy");
    }
}
```

我們就在另一個元件中使用剛剛所制定的 `Jimmy` 這個元件，並且將資料輸出在畫面上。

```razor
@page "/"

<Jimmy @bind-Value="data" />
<span>hi @data</span>

@code {
  string data;
}
```

執行以後應該可以看到像下面的結果，雙向綁定的行為是成功的。
![two-way-binding](two-way-binding.gif)

## 多層雙向綁定

接著我們要寫一個元件叫做 **TextField.razor**，裡面會包含 label 以及 input，並且希望能夠直接將 input 的資料做雙向綁定。

如果只照著前面的做法來做，會遇到一個問題，input 本身有雙向綁定，會需要一個屬性來承接，但又要能直接跟開放出去的屬性作連接，這時可以使用一個新的屬性來做關聯。

```razor
<label>
    @Text
    <input @bind="Data" />
</label>

@code {
    [Parameter] public string Text { get; set; }
    [Parameter] public string Value { get; set; }
    [Parameter] public EventCallback<string> ValueChanged { get; set; }
    string Data {
    	get => Value;
    	set => ValueChanged.InvokeSync(value);
    }
}
```

# 參數

## 未定義參數

如果是很明確所要開放的參數，通常我們會直接定義屬性，但 html 原生的屬性，就不會想要用這樣的方式一個一個去做開放，我們可以將 **ParameterAttribute** 的屬性 `CaptureUnmatchedValues` 設定為 true，就可以拿到剩下未定義但外面有傳的參數。

```razor
<div @attributes="Attributes" id="test" />

@code {
    [Parameter(CaptureUnmatchedValues = true)]
	public IReadOnlyDictionary<string, object> Attributes { get; set; }
}
```

```html
<TextField id="jimmy" />
```

這邊可以看到輸出的 id 會是 *test* 而不是 *jimmy* ，要特別注意的是定義的順序，會影響到輸出的結果，如果希望以外面定義的為主，那就要把 `@attributes` 放在最後面。

## 重新組合 class

元件通常會有自己的樣式，也會提供給外面覆寫 class，我們可以透過上一個方法所定義的屬性，來取得外面所定義的 class。

```razor
<div class="@GetClass()" />

@code {
    [Parameter(CaptureUnmatchedValues = true)]
	public IReadOnlyDictionary<string, object> Attributes { get; set; }
	
	string GetClass()
	{
		var baseClass = "d-flex flex-column";
		if (Attributes != null && Attributes.TryGetValue("class", out var @class) && !string.IsNullOrEmpty(Convert.ToString(@class)))
		{
			return $"{baseClass} {@class}";
		}
		
		return baseClass;
	}
}
```

# 使用樣板

在寫元件時，為了要開放給使用者決定部分區的內容，我們可以透過開放樣板屬性的方式來達成， **RenderFragment** 這個類別可以來承接 html 的內容。

## 使用預設名稱

只有屬性名稱叫做 `ChildContent` ，使用端才能省略不用屬性名稱包住內容。

```razor
<!-- JimmyTemplate.razor -->
<div class="button-wrapper">
	@ChildContent
</div>

@code {
	[Parameter] public RenderFragment ChildContent { get; set; }
}
```

```html
<JimmyTemplate>
	<button>confirm</button>
	<button>cancel</button>
</JimmyTemplate>
```

## 非預設名稱

不管有幾個樣板，只要屬性名稱不叫 `ChildContent` ，使用端都要使用屬性名稱包住。

```razor
<!-- JimmyTemplate.razor -->
<div class="button-wrapper">
	@CustomWrapper
</div>

@code {
	[Parameter] public RenderFragment CustomWrapper { get; set; }
}
```

```html
<JimmyTemplate>
	<CustomWrapper>
		<button>confirm</button>
		<button>cancel</button>
	</CustomWrapper>
</JimmyTemplate>
```

# 參考

* [ASP.NET Core Razor components | Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/blazor/components/?view=aspnetcore-5.0)
* [radzenhq/radzen-blazor: The home of the Radzen Blazor components. (github.com)](https://github.com/radzenhq/radzen-blazor)