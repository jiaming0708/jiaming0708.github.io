---
title: 在UpdatePanel裡執行JavaScript
date: 2011-10-05 09:27:52
categories:
- Back-end
- ASP.NET
tags:
- ASP.NET
---
昨天遇到一個問題，要在UpdatePanel裡面的某個物件PostBack後執行Script
一般在按鈕觸發後回到後台執行完畢，要在執行某個Script
我們會這樣寫

``` js
ClientScript.RegisterStartupScript(this.GetType(), "ShowMessage", "alert('hello!')");
```

但是，如果按鈕被包在UpdatePanel裡面
這樣的寫法無效，並不會執行
這時我們換一個寫法，就可以正常執行了

``` js
ClientScript.RegisterClientScriptBlock(UpdatePanel1.GetType(), "ShowMessage", "alert('hello!')", true);
```