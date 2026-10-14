---
title: '[DotnetCore] Web API 直接 serve 靜態 HTML/CSS'
date: 2026-10-14 20:02:59
updated: 2026-10-14 20:02:59
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
thumbnail: /images/dotnet-core.png
---

幾個月前 PM 和設計師突然想到有一些行銷或是廣告頁面，能不能由他們提供完整的 html/js/css，然後直接掛到 server。一直都知道 Dotnet Core 可以這樣做，但有一些眉角要特別注意，不然容易採雷。

<!-- more -->

## 使用 Static Files middleware

在 `Program.cs` 加兩行，再把整包檔案放進 `wwwroot/`。

1. `Program.cs`

>Web API 專案預設沒有開，順序不能錯

```csharp
var app = builder.Build();

app.UseDefaultFiles();   // 目錄請求自動對應 index.html
app.UseStaticFiles();    // serve wwwroot/ 底下的靜態檔

app.MapControllers();    // 原本的 API 路由不動
app.Run();
```

2. 檔案放進 `wwwroot/`

```
YourProject/
├── wwwroot/
│   ├── index.html
│   ├── css/style.css
│   ├── js/main.js
│   └── images/logo.png
├── Controllers/
└── Program.cs
```

`wwwroot/` 預設就會隨發佈一起打包，`.csproj` 通常不用額外設定。之後改 CSS、JS、圖片或新增 HTML，只要替換檔案並重新部署，不需要動任何 C# 程式。

## 實際的請求路徑

以廣告頁來說，假設放在 `wwwroot/event_A/index.html`，那對外的網址會是這樣：

| 請求路徑 | 對應檔案 | 說明 |
| --- | --- | --- |
| `/event_A/` | `wwwroot/event_A/index.html` | 由 `UseDefaultFiles` 幫忙補上 index.html |
| `/event_A/index.html` | `wwwroot/event_A/index.html` | 直接指定檔名，一定有效 |
| `/event_A`（沒有結尾斜線） | 同上 | 通常會 redirect 到 `/event_A/`，多一次跳轉 |

所以廣告連結我會直接給 `/event_A/`，少一次 redirect。

## 踩雷：`/event_A/` 回 404

如果照上面設定完，打 `/event_A/` 卻是 404，可以依序檢查以下幾點：

1. **順序**：`UseDefaultFiles` 一定要在 `UseStaticFiles` 之前，這是最常發生的狀況。
2. **大小寫**：在 Windows 開發不分大小寫，但部署到 Linux 或是容器就會區分，`event_A` 和 `event_a` 是不同路徑。
3. **`wwwroot` 有沒有被認到**：印出 `builder.Environment.WebRootPath` 確認，如果是 `null` 或路徑不對，就要手動指定：

```csharp
builder.Environment.WebRootPath =
    Path.Combine(Directory.GetCurrentDirectory(), "wwwroot");
```

4. **看實際的 HTTP 狀態碼**，可以縮小範圍：

| 狀態碼 | 通常代表 |
| --- | --- |
| 404 | 找不到檔案，多半是路徑、大小寫或順序問題 |
| 403 | 找到目錄但不給列出內容，代表 `UseDefaultFiles` 沒有生效 |
| 301/302 | 被導到別的地方 |

## 為什麼順序很重要

`UseDefaultFiles` 是 `IApplicationBuilder` 的擴充方法，背後註冊的是 `DefaultFilesMiddleware`，這邊很容易誤會成它會回傳檔案，其實**它只負責改寫請求路徑**，真正回傳檔案的是 `UseStaticFiles`。

流程大概是這樣：

1. 收到請求 `/event_A/`
2. middleware 判斷這是目錄，而且 `wwwroot/event_A/` 底下有預設檔
3. 在 pipeline 內部把 `Request.Path` 改寫成 `/event_A/index.html`，這是內部改寫，不是 redirect，瀏覽器的網址不會變
4. 交給下一個 middleware `UseStaticFiles` 把檔案回傳

既然是先改寫、後回傳，那順序反過來的話，`UseStaticFiles` 先處理了 `/event_A/`，找不到檔案就直接 404，後面的改寫也就沒意義了。

預設會依序找 `default.htm`、`default.html`、`index.htm`、`index.html`，如果檔名不一樣也可以自己指定：

```csharp
var options = new DefaultFilesOptions();
options.DefaultFileNames.Clear();
options.DefaultFileNames.Add("main.html");
app.UseDefaultFiles(options);
app.UseStaticFiles();
```

## 順序改正會影響其他功能嗎

如果原本專案已經是 `UseStaticFiles` 在前面，想改成正確的順序，會不會影響到其他功能？不會，唯一的差別就是 `/event_A/` 這種省略檔名的路徑開始有作用而已。

| 請求 | 改之前（`UseStaticFiles` 在前） | 改之後（`UseDefaultFiles` 在前） |
| --- | --- | --- |
| `/event_A/` | 404 | 對應 `index.html` |
| `/event_A/index.html` | 正常 | 正常 |
| `.css`／`.js`／圖片 | 正常 | 正常 |
| `/api/...` 路由 | 不受影響 | 不受影響 |

因為 `UseDefaultFiles` 只處理「目錄的請求」，不會碰到 `MapControllers` 的 API 路由，也不會影響已經有明確檔名的靜態檔，所以可以放心調整。

## 心得

整理一下需要注意的地方：`UseDefaultFiles` 要在 `UseStaticFiles` 前面、部署到 Linux 要注意路徑的大小寫，另外 API 最好統一加上前綴（例如 `/api`），避免和靜態檔的路徑衝突。
