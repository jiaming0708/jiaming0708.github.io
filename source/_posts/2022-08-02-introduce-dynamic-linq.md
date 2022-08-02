---
title: '[c#] EF使用動態LINQ做查詢'
date: 2022-08-02 14:43:07
categories:
- Backend
- linq
tags:
- dotnet
- linq
---

LINQ 是一個好用的工具，撰寫的時候可以透過強行別的特性，讓篩選資料等的行為很輕鬆，偏偏成也強型別敗也強型別，在做查詢的時候十幾個欄位，總不可能一個一個比對，還好查到一個好用的套件 [Dynamic-LINQ]([Dynamic LINQ (dynamic-linq.net)](https://dynamic-linq.net/))，可以解決這樣的問題。

<!-- more -->

先來建立一個簡單的客戶類別，並產生一些資料

```c#
public class Customer
{
	public int CustomerId;
	public string FirstName;
	public string LastName;
	public string Company;
}

var data  = new List<Customer>()
{
    new Customer()
    {
        CustomerId = 1,
        FirstName = "Jimmy",
        LastName = "Ho",
        Company = "Jimmy Company",
    },
    new Customer()
    {
        CustomerId = 2,
        FirstName = "Secret",
        LastName = "Ho",
        Company = "Secret note",
    },
    new Customer()
    {
        CustomerId = 3,
        FirstName = "Tina",
        LastName = "Hu",
        Company = "Tina Company",
    }
};
```

標準 Linq 的寫法，取得 `Company` 包含 `Compnay` 字樣的資料。

```c#
var result = data.Where(p => p.Company.Contains("Ho"));
```

接著來安裝 Dynamic LINQ，在 Nuget 中找到 **System.Linq.Dynamic.Core**，並改用 Dynamic 的寫法。

> 如果不是 dotnet core 請直接搜尋 System.Linq.Dynamic

```c#
var result = data.Where("Company.Contains(\"Ho\")");
```

透過字串的方式，來描述原本強型別的語句，當欄位是由前端傳來只有名稱，這時候就會很好處理。
使用 Dynamic LINQ 要注意的是，要參照原本 Linq 的寫法，不然就會死得不明不白。

> 當然這也不是沒有缺點，因為不是強型別了，所以屬性名稱寫錯的話，就要等到 runtime 才會死在旁邊。

# Reference

- [[LINQ\] 使用 Dynamic LINQ 補足強型別所帶來的小遺憾 | 搞搞就懂 - 點部落 (dotblogs.com.tw)](https://dotblogs.com.tw/wasichris/2015/12/17/001236)

- [Dynamic LINQ 讓 LINQ 的世界變的更美好 | The Will Will Web (miniasp.com)](https://blog.miniasp.com/post/2008/06/23/Introduce-LINQ-Dynamic-Query-Library)