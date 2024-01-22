---
title: '[Blazor] 傳遞方法為參數'
date: 2024-01-22 14:37:19
updated: 2024-01-22 14:37:19
categories:
- Frontend
- Blazor
tags:
- Blazor
thumbnail: /images/blazor.jpg
---

Blazor 的參數基本上可以分為事件外拋和傳值進去，但有時候會希望子元件能去執行母元件的某個方法，這個時候可以使用 c# 的 Delegate 的行為來實作。

<!-- more -->

## 使用 Event

一般的情況下，會將 Parameter 宣告為 EventCallback 來作為跟外面元件做溝通

> Parent compoennt

```html
<div>
  <Child count="@count" countChange="@OnChange"/>
</div>

@code
{
  private int count;
  private void OnChange(int val)
  {
    count = val;
  }
}
```

> Child component

```html
<div>
    <span>@count</span>
    <button @onclick="@OnClick">
        click me
    </button>
</div>

@code
{
  [Parameter] public int count { get; set; }
  [Parameter] public EventCallback<int> countChange { get; set; }
      
  private async Task OnClick()
  {
      await countChange.InvokeAsync(count+1);
  }
}
```

## 使用 Delegate

如果希望把一些運算是由母元件來決定，子元件只管畫面和基本的行為，那就可以使用 Delegate 的做法。

以上面的例子來說，將 count 的計算交給外面，裡面只做顯示數字和按鈕的事件。

> 子元件

```razor
<div>
    <span>@count</span>
    <button @onclick="@OnClick">
        click me
    </button>
</div>

@code
{
  [Parameter] public int count { get; set; }
  [Parameter] public Func<int, int> countChange { get; set; }
      
  private async Task OnClick()
  {
    var result = countChange(count);
    Console.WriteLine($"parameter count: {count}, result {result}");
  }
}
```

> 母元件

```
<div>
  <Child count="@count" countChange="@OnChange"/>
</div>

@code
{
  private int count;
  private int OnChange(int val)
  {
    count = val + 1;
    StateHasChanged();
    return count;
  }
}
```

> 這邊要注意 `StateHasChanged` ，才能讓 blazor 重新 render 子元件。

根據這樣的修改，也就是說，可以讓把相同畫面的功能抽成元件，細節都交給外面，像是下面一次 `+2` 的功能。

```razor
<div>
  <Child count="@count" countChange="@OnChange"/>
</div>

@code
{
  private int count;
  private int OnChange(int val)
  {
    count = val + 2;
    StateHasChanged();
    return count;
  }
}
```

也許你會問那有機會用到 `Action<>` 嗎? 個人認為不太可能，因為 Action 是沒有回傳值，依照這種情境就直接用 `EventCallback` 更好。