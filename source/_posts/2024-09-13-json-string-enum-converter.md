---
title: '[Dotnet] JsonStringEnumConverter 的應用'
date: 2024-09-13 19:32:22
updated: 2024-09-13 19:32:22
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
thumbnail:
---

最近在寫需求時，前端回報 API 的某個欄位不是應該數字 enum 嗎，怎麼變成文字，檢查以後才發現，前人使用 `JsonStringEnumConverter` 這個功能將 enum 全部轉成 string 輸出給前端。

為了要讓舊有 API 維持，新的 API 也可以照所希望的輸出成數字，而去找了一些解法。

<!-- more -->

## 統一輸出格式

如果想要全站的 Enum 都輸出的時候都是 string 的格式，可以在 `JsonSerializerOptions` 加上 `JsonStringEnumConverter` 設定。

```c#
// controller 寫法
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.Converters.Add(new JsonStringEnumConverter());
    });

// minial api 寫法
builder.Services.Configure<JsonOptions>(options => options.SerializerOptions.Converters.Add(new JsonStringEnumConverter()));
```

> 這邊要注意，要根據你的寫法來使用不同的設定方法，給 minial api 的寫法沒有相容 controller 的設定。

## Ignore 部分 Enum

前面的做法是統一輸出格式都是 string，如果今天有一個新的 enum 並不希望被輸出成 string，那可以怎麼辦?

首先要搞懂 `JsonStringEnumConverter` 到底做了些什麼，根據 source code 可以看到 `CanConvert` 這個方法內做了 `IsEnum` 的判斷

{% ghcode https://github.com/dotnet/runtime/blob/main/src/libraries/System.Text.Json/src/System/Text/Json/Serialization/JsonStringEnumConverter.cs 64 121 %}

根據這個做法，可以來客製一個 Converter 讓指定的 enum 可以忽略這個規則

> 參考自 [stackoverflow](https://stackoverflow.com/a/59832092)，這個做法可以忽略多種 Converter

```c#
public class IgnoreJsonConverterFactory : JsonConverterFactory
{
    private readonly JsonConverterFactory _innerFactory;
    readonly HashSet<Type> optOutTypes;

    public IgnoreJsonConverterFactory(JsonConverterFactory innerFactory, params Type [] optOutTypes)
    {
        _innerFactory = innerFactory;
        this.optOutTypes = optOutTypes.ToHashSet();
    }

    public override bool CanConvert(Type typeToConvert)
    {
        return _innerFactory.CanConvert(typeToConvert) && !optOutTypes.Contains(typeToConvert);
    }
    public override JsonConverter CreateConverter(Type typeToConvert, JsonSerializerOptions options)
    {
        return _innerFactory.CreateConverter(typeToConvert, options);
    }
}
```

在 Converter 設定時就可以指定要忽略規則的 enum

```c#
options.SerializerOptions.Converters.Add(new OptOutJsonConverterFactory(new JsonStringEnumConverter(),
    typeof(DayOfWeek)
));
```

> 這個做法可以優化成使用 attribute

## 指定 Enum 輸出

當 Enum 都是自己定義，那也可以換個思維，指定哪些 enum 輸出成 string，`JsonConverter` 本身可以當作一個 attribute 來使用
有設定這些 Attribute 的在輸出時，都會根據設定來做轉換，連 service 都不用設定，更加優美

> [How to customize property names and values with System.Text.Json - .NET | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/customize-properties?pivots=dotnet-8-0#enums-as-strings)

```c#
[JsonConverter(typeof(JsonStringEnumConverter<Precipitation>))]
public enum Precipitation
{
    Drizzle, Rain, Sleet, Hail, Snow
}
```

