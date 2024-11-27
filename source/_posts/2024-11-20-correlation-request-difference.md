---
title: 'X-Correlation-Id 與 X-Request-Id 差異介紹'
date: 2024-11-20 21:35:54
updated: 2024-11-20 21:35:54
categories:
- Backend
- Infrastructure
tags:
- DotnetCore
- Infrastructure
thumbnail:
---

在現代分散式系統中，當有多個服務共同協作來處理單一請求時，對於整個請求的追蹤與調試變得至關重要。這時，**X-Correlation-Id** 和 **X-Request-Id** 這些 HTTP 標頭能提供幫助。

> 本篇由 ChatGPT 產生
> 程式片段測試有效，可安心服用

<!-- more -->

### X-Correlation-Id 與 X-Request-Id 的差異

#### **X-Correlation-Id**

- **用途**：用於跨系統的請求追蹤。
- **範圍**：通常用於**多個服務之間**，確保每個服務在處理同一請求的過程中可以共用相同的識別碼。
- **典型場景**：分散式系統中，記錄一整條請求的處理鏈路（例如 A -> B -> C）。
- 設計重點：
  - 通常由請求的**初始服務**產生，並在所有下游服務中傳遞。
  - 使用相同的 ID 作為一個“上下文標識符”。

#### **X-Request-Id**

- **用途**：用於單一請求的追蹤。
- **範圍**：單個服務內部的請求標識。
- **典型場景**：在某一個微服務內，追蹤該服務接收到的每個請求。
- 設計重點：
  - 每個請求都會生成一個唯一的 `Request-Id`。
  - 即便是相同的 Correlation-Id，Request-Id 也會是唯一的，代表該請求的服務內處理實例。

### 使用時機

1. **X-Correlation-Id**：
   - 用於跨多個服務之間的請求追蹤。
   - 例如，在微服務架構中，當請求從前端到後端，並經過多個微服務時，X-Correlation-Id 可以幫助你關聯這些請求。
   - 適用於分佈式追蹤與集中化日誌查詢工具，如 ELK 或 Jaeger。
2. **X-Request-Id**：
   - 用於服務內的請求識別。
   - 例如，一個服務可能會處理來自多個客戶端的請求，而每個請求都有不同的 X-Request-Id，方便區分。
   - 適用於單個服務內的細粒度調試與問題定位。

### Dotnet Core 實作範例

以下是用 ASP.NET Core 8 實現 **X-Correlation-Id** 和 **X-Request-Id** 的範例，包括如何在中介軟體（Middleware）中處理它們。

#### 1. **定義 Middleware**

我們需要建立一個中介軟體來處理 `X-Correlation-Id` 和 `X-Request-Id`：

```c#
public class CorrelationIdMiddleware
{
    private readonly RequestDelegate _next;

    public CorrelationIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // 檢查是否有傳入 X-Correlation-Id，否則生成一個新的
        if (!context.Request.Headers.TryGetValue("X-Correlation-Id", out var correlationId))
        {
            correlationId = Guid.NewGuid().ToString();
            context.Request.Headers["X-Correlation-Id"] = correlationId;
        }

        // 確保在 Response Header 中返回 X-Correlation-Id
        context.Response.Headers["X-Correlation-Id"] = correlationId;

        // 為每個請求生成唯一的 X-Request-Id
        var requestId = Guid.NewGuid().ToString();
        context.Request.Headers["X-Request-Id"] = requestId;
        context.Response.Headers["X-Request-Id"] = requestId;

        // 將 CorrelationId 和 RequestId 傳遞給下游處理
        await _next(context);
    }
}
```

#### 2. **註冊 Middleware**

將中介軟體註冊到應用程序中：

```c#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 註冊 CorrelationIdMiddleware
app.UseMiddleware<CorrelationIdMiddleware>();

app.MapGet("/", (HttpContext context) =>
{
    var correlationId = context.Request.Headers["X-Correlation-Id"].ToString();
    var requestId = context.Request.Headers["X-Request-Id"].ToString();

    return Results.Json(new
    {
        Message = "Hello, world!",
        CorrelationId = correlationId,
        RequestId = requestId
    });
});

app.Run();
```

- ### 延伸功能

  1. **集中式日誌追蹤**：
     - 使用 `X-Correlation-Id` 串聯多個服務的日誌，將其輸出到工具（如 ELK、Jaeger）。
     - 在 ASP.NET Core 中，將 Correlation-Id 嵌入日誌中：

```c#
using Microsoft.Extensions.Logging;

public class CorrelationLoggerMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<CorrelationLoggerMiddleware> _logger;

    public CorrelationLoggerMiddleware(RequestDelegate next, ILogger<CorrelationLoggerMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = context.Request.Headers["X-Correlation-Id"].ToString();
        using (_logger.BeginScope(new Dictionary<string, object> { { "CorrelationId", correlationId } }))
        {
            await _next(context);
        }
    }
}
```

**錯誤追蹤與回報**：

- 在 API 錯誤回應中返回 `X-Correlation-Id`，讓使用者可以回報問題時提供更準確的追蹤資訊。

### 結論

**X-Correlation-Id** 和 **X-Request-Id** 是分散式系統中不可或缺的請求追蹤工具：

- **X-Correlation-Id** 用於關聯跨多個服務的請求。
- **X-Request-Id** 用於區分單一服務內的請求。

透過 ASP.NET Core 的中介軟體，我們能夠輕鬆地實現這兩者，並將其整合到集中式日誌系統或錯誤追蹤系統中，從而提升系統的可觀察性與調試能力。