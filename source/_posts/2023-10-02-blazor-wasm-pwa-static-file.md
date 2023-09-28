---
title: '[Blazor] PWA 靜態檔案問題'
date: 2023-10-02 14:13:19
updated: 2023-10-02 14:13:19
categories:
- Frontend
- Blazor
tags:
- Blazor
thumbnail: /images/blazor.jpg
---

在 Blazor WebAssembly 中可以開啟 PWA 的模式，也就是會產生 Service Worker，要加入像是 Excel 或是 PDF 到靜態檔案，會發現在本機開發時都能夠正常下載的檔案，怎麼上到掛 domain 的 server 就會去走 routing，跟預期的差很多，今天會來解析中間發生了什麼事情。

<!-- more -->

> 以下版本為 Dotnet 7.0.4

## Reproduce

### 專案建置

先來試著模擬一下情境，為了更明確的知道其中的差異性，接下來會需要建置兩個專案（或是同一個專案改成 PWA）。

```shell
# 沒有 PWA
dotnet new blazorwasm -n sample1
# 有 PWA
dotnet new blazorwasm --pwa -n sample2
```

到[人事行政局](https://www.dgpa.gov.tw/information?uid=41&pid=11397)下載 2024 年（民國 113 年）的行事曆，Excel 或是 PDF 都可以。
把檔案放到 wwwroot 底下，並且在 `Index.razor` 加上一個下載的連結。

```html
@page "/"

<PageTitle>Index</PageTitle>

<h1>Hello, world!</h1>

Welcome to your new app.

<!-- 增加連結 -->
<div>
  <a href="113年辦公日曆表.pdf">台灣 2024 年行事曆</a>
</div>

<SurveyPrompt Title="How is Blazor working for you?" />

```

兩個專案都要做一次同樣的事情，並且用 `dotnet run` 來確認結果。

![sample1-local-run](sample1-local-run.png)

![sample2-local-run](sample2-local-run.png)

### 模擬server

接著要來模擬 deploy 到 server 後的情境，搭配使用 [ngrok](https://ngrok.com/download) 工具來實現 domain 的情境。
執行 `publish` 的指令，並且將輸出目錄指定為 `Release`。

> 後面再來說為什麼要用這樣的模式

```shell
dotnet publish -c Release -o Release
cd ./Release/wwwroot
# 透過 npm 套件 http-server 來架網站，如果是 windows 也可以用 IIS 來做到相同的事情
npx http-server
```

![npx-http-server](npx-http-server.png)

先執行看看，確保網站能夠正確的呈現，接著要啟動 ngrok 來做 reverse proxy。

```
ngrok http 8080
```

![ngrok-run](ngrok-run.png)

依序執行兩個專案，記得到 sample2 的時候要強制 refresh 才會吃到正確的內容。

> 簡單一點也可以是關掉 ngrok 重新執行，會取得新的 domain

用 domain 開啟 sample2 網站，試著點擊行事曆的連結，就會發現到系統竟然是重新載入頁面並且說找不到路由。

![pwa-issue-reproduce](pwa-issue-reproduce.png)

##Root Cause

使用 localhost 開啟不管是不是 PWA 網站都沒事，只要用 Domain 開啟 PWA 就會發生路由問題，這到底是為什麼？

### PWA

按照前面來看最大問題點應該是 PWA 及 Domain，那就先來看看微軟如何[介紹 PWA](https://learn.microsoft.com/en-us/aspnet/core/blazor/progressive-web-app?view=aspnetcore-7.0&tabs=visual-studio-code)，並且搞清楚有沒有 PWA 的專案差異在哪。

- Working offline and loading instantly, independent of network speed.
- Running in its own app window, not just a browser window.
- Being launched from the host's operating system start menu, dock, or home screen.
- Receiving push notifications from a backend server, even while the user isn't using the app.
- Automatically updating in the background.

跟一般的網頁最大差異有兩個，第一個是可以**離線運作**，第二個是可以**安裝**。
接著要來看看差異是什麼，從檔案的差異來看，可以看到 wwwroot 底下多了兩個檔案，`service-worker.js` 及 `service-worker.published.js`。

![pwa-wwwroot-diff](pwa-wwwroot-diff.png)

`csproj` 裡面的設定也有兩個地方不同，都是跟 `service-worker` 有關係。

```xml
<Project Sdk="Microsoft.NET.Sdk.BlazorWebAssembly">

  <PropertyGroup>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <!-- 新增設定 -->
    <ServiceWorkerAssetsManifest>service-worker-assets.js</ServiceWorkerAssetsManifest>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Components.WebAssembly" Version="7.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Components.WebAssembly.DevServer" Version="7.0.0" PrivateAssets="all" />
  </ItemGroup>

  <ItemGroup>
    <!-- 新增設定 -->
    <ServiceWorker Include="wwwroot\service-worker.js" PublishedContent="wwwroot\service-worker.published.js" />
  </ItemGroup>

</Project>
```

### Service Worker

既然重點會是在 Service Worker，那就來看看 [MDN 的介紹](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker)。

> **Secure context:** This feature is available only in [secure contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts) (HTTPS), in some or all [supporting browsers](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker#browser_compatibility).

在最開頭就有說到，這個功能只會在 `HTTPS` 的環境下啟動，那這樣也能夠解釋為什麼 localhost 執行會是正常。
接著的問題就會是，為什麼會去走路由，而不是直接下載檔案。分別來看看兩個 `service-worker.*.js` 裡面做了什麼事情。

#### service-worker.js

在開發模式下，就會是這個檔案內容，可以看到的是根本沒有關於到 service worker 的功能，因此在開發階段怎麼樣的測試都會是正常的。

```javascript
// In development, always fetch from the network and do not enable offline support.
// This is because caching would make development more difficult (changes would not
// be reflected on the first load after each change).
self.addEventListener('fetch', () => { });
```

#### service-woker.published.js

發行過後，dotnet 編譯會將這個檔案內容取代原本的 `service-worker.js`，底下就試著去搞懂這段邏輯在做些什麼。

```javascript
// Caution! Be sure you understand the caveats before publishing an application with
// offline support. See https://aka.ms/blazor-offline-considerations

self.importScripts('./service-worker-assets.js');
self.addEventListener('install', event => event.waitUntil(onInstall(event)));
self.addEventListener('activate', event => event.waitUntil(onActivate(event)));
self.addEventListener('fetch', event => event.respondWith(onFetch(event)));

const cacheNamePrefix = 'offline-cache-';
const cacheName = `${cacheNamePrefix}${self.assetsManifest.version}`;
const offlineAssetsInclude = [ /\.dll$/, /\.pdb$/, /\.wasm/, /\.html/, /\.js$/, /\.json$/, /\.css$/, /\.woff$/, /\.png$/, /\.jpe?g$/, /\.gif$/, /\.ico$/, /\.blat$/, /\.dat$/ ];
const offlineAssetsExclude = [ /^service-worker\.js$/ ];

async function onInstall(event) {
    console.info('Service worker: Install');

    // Fetch and cache all matching items from the assets manifest
    const assetsRequests = self.assetsManifest.assets
        .filter(asset => offlineAssetsInclude.some(pattern => pattern.test(asset.url)))
        .filter(asset => !offlineAssetsExclude.some(pattern => pattern.test(asset.url)))
        .map(asset => new Request(asset.url, { integrity: asset.hash, cache: 'no-cache' }));
    await caches.open(cacheName).then(cache => cache.addAll(assetsRequests));
}

async function onActivate(event) {
    console.info('Service worker: Activate');

    // Delete unused caches
    const cacheKeys = await caches.keys();
    await Promise.all(cacheKeys
        .filter(key => key.startsWith(cacheNamePrefix) && key !== cacheName)
        .map(key => caches.delete(key)));
}

async function onFetch(event) {
    let cachedResponse = null;
    if (event.request.method === 'GET') {
        // For all navigation requests, try to serve index.html from cache
        // If you need some URLs to be server-rendered, edit the following check to exclude those URLs
        const shouldServeIndexHtml = event.request.mode === 'navigate';

        const request = shouldServeIndexHtml ? 'index.html' : event.request;
        const cache = await caches.open(cacheName);
        cachedResponse = await cache.match(request);
    }

    return cachedResponse || fetch(event.request);
}
```

首先，看到 import  一個額外的檔案 `service-worker-assets.js`，但在 source code 裡面沒找到，可以先推測是編譯的時候才產生。往下看到 `offlineAssetsInclude` 這個變數，宣告各種常見的靜態檔案，像是圖片或是字形檔；`offlineAssetsExclude` 變數則是把 `service-worker.js` 宣告為不可快取，畢竟 service-worker 一旦被快取，那 server 有更新的話就會沒跟上。
底下的三個方法，就是決定哪些請求是可以直接走快取，哪些則是要去跟 server 做請求。

##### onInstall

當 service worker 安裝的時候，會把 `service-worker-assets.js` 檔案中設定且符合 `offlineAssetsInclude` 的檔案加入快取清單。

##### onActivate

清除掉已經無效的快取。

##### onFetch

請求是 GET 的情況下，會檢查請求路徑是否符合快取的檔案，如果是的話則直接取得檔案；其他則是直接跟 server 做請求。

#### service-worker-assets.js

如前面所說，這個檔案是編譯才會有的，在 sample2/Release 資料夾找到該檔案。
檔案結構很簡單，就是 `assetsManifest` 這個物件中有兩個屬性 `assets` 及 `version`。可以在 assets 中看到 wwwroot 底下的檔案幾乎都有被放進去，唯獨我們這次的目標 `113年辦公日曆表.pdf` 並沒有在清單中，依照前面的 `onFetch` 邏輯，這時候自然會直接跟 server 做請求也就是會走路由的行為，並且回報路由錯誤。

```javascript
self.assetsManifest = {
  "assets": [
    {
      "hash": "sha256-gMMhXf930MHxSiJZcjP4DGT41x\/\/4UluuuaVRM2SZIk=",
      "url": "_framework\/blazor.webassembly.js"
    },
    {
      "hash": "sha256-JmMF+3r0RjsO0yYBfDP8Pq2ahxBSJHFu7mZNB9lqLsw=",
      "url": "_framework\/dotnet.7.0.7.lvcvwsp4ai.js"
    },
    {
      "hash": "sha256-E5nX3n\/\/2kvaYqbQ2VjaS4BXnlnEYW3gU0pr58g11a4=",
      "url": "_framework\/dotnet.timezones.blat"
    },
    {
      "hash": "sha256-q7mSjRyJTT10PpQnSxO1TZAwei3YpzD76Akn6E57GTU=",
      "url": "_framework\/dotnet.wasm"
    },
    {
      "hash": "sha256-SZLtQnRc0JkwqHab0VUVP7T3uBPSeYzxzDnpxPpUnHk=",
      "url": "_framework\/icudt_CJK.dat"
    },
    {
      "hash": "sha256-8fItetYY8kQ0ww6oxwTLiT3oXlBwHKumbeP2pRF4yTc=",
      "url": "_framework\/icudt_EFIGS.dat"
    },
    {
      "hash": "sha256-L7sV7NEYP37\/Qr2FPCePo5cJqRgTXRwGHuwF5Q+0Nfs=",
      "url": "_framework\/icudt_no_CJK.dat"
    },
    {
      "hash": "sha256-tO5O5YzMTVSaKBboxAqezOQL9ewmupzV2JrB5Rkc8a4=",
      "url": "_framework\/icudt.dat"
    },
    {
      "hash": "sha256-1+1o3tb7DjOeVSY3fkfvtXGL8VrnvSojyRwqtP0L7Ms=",
      "url": "_framework\/blazor.boot.json"
    },
    {
      "hash": "sha256-h3b\/h8RHSoCJ0Sjd6yf8VfoKk4va5PcIDZr\/COzisyk=",
      "url": "_framework\/Microsoft.AspNetCore.Components.dll"
    },
    {
      "hash": "sha256-M4eAEQfmoZY9Hoblq1fGL6MCo\/dlBEjHR0mgArTQH1U=",
      "url": "_framework\/Microsoft.AspNetCore.Components.Web.dll"
    },
    {
      "hash": "sha256-tpvXX7M\/LEGTEvYWgu8LEXMhjtjwoFAKuxUiqdpIH2E=",
      "url": "_framework\/Microsoft.AspNetCore.Components.WebAssembly.dll"
    },
    {
      "hash": "sha256-X\/f4fDl2cuIRXeWHhK\/f2UqQbFioD+RU4a4CEh0zrrQ=",
      "url": "_framework\/Microsoft.Extensions.Configuration.Abstractions.dll"
    },
    {
      "hash": "sha256-DBOKSPriP2JDxVbbWrLXyD3K4\/x3RBifNBWk\/q1I39M=",
      "url": "_framework\/Microsoft.Extensions.Configuration.dll"
    },
    {
      "hash": "sha256-Q5AqJneA2TZnzC0IYzBx6j\/tHRhWAeMbpH3BsV7KgWg=",
      "url": "_framework\/Microsoft.Extensions.Configuration.Json.dll"
    },
    {
      "hash": "sha256-7ni+nO8H2QxkXyRXRureFzJc0Tr5aT45TAYcdlgpmwQ=",
      "url": "_framework\/Microsoft.Extensions.DependencyInjection.Abstractions.dll"
    },
    {
      "hash": "sha256-qi0kE7rp0kdsNqdL6DyPZEeimjUGvcLT4iWQX0YnRus=",
      "url": "_framework\/Microsoft.Extensions.DependencyInjection.dll"
    },
    {
      "hash": "sha256-qZdDWEYBno4I8vPywu9Fyn5LLwSVquG8Y7JzRiWgFEA=",
      "url": "_framework\/Microsoft.Extensions.Logging.Abstractions.dll"
    },
    {
      "hash": "sha256-RJE660TEC1wm3MyL1EXQfTQB1qK+JWy1efVUxI8nNUo=",
      "url": "_framework\/Microsoft.Extensions.Logging.dll"
    },
    {
      "hash": "sha256-QqyagFWSuupxyjIZM2m\/ovuMuvYHTCjR2+BRNiUT4WA=",
      "url": "_framework\/Microsoft.Extensions.Options.dll"
    },
    {
      "hash": "sha256-eXvGx2jcjpTPEJoAHBsW\/VuMPbNyyU+AsuhPmkzSSRY=",
      "url": "_framework\/Microsoft.Extensions.Primitives.dll"
    },
    {
      "hash": "sha256-O2v\/TEzGxRkJx2s8BIzjplzaBZGIjAEU8QLVvMi29Pk=",
      "url": "_framework\/Microsoft.JSInterop.dll"
    },
    {
      "hash": "sha256-6yDYSBiy48fNaYJlqK10GSN72A6Vsu3i\/6v3Lq7kL2E=",
      "url": "_framework\/Microsoft.JSInterop.WebAssembly.dll"
    },
    {
      "hash": "sha256-QV+RIj4HlZ0kGZ2\/Z7+3KCHKBuuBWY4sQq+3Tuu4G+g=",
      "url": "_framework\/sample2.dll"
    },
    {
      "hash": "sha256-M5QNIjWTWuKn8+IKAVWEdjk7OB2HhnQd\/jmEz0YFfTQ=",
      "url": "_framework\/System.Collections.Concurrent.dll"
    },
    {
      "hash": "sha256-yNgdj0HVTyDrEY9f0Y9on7p6DfvN2\/Av1guJChKeYMI=",
      "url": "_framework\/System.Collections.dll"
    },
    {
      "hash": "sha256-aF2ysxa8maJxS6cn0NMTkjFFkQQ2EGqlsZXcjbHKHAo=",
      "url": "_framework\/System.ComponentModel.dll"
    },
    {
      "hash": "sha256-UX+onNe4XrzDF4D77LZbU62NxGG584NgorjUWc44IuA=",
      "url": "_framework\/System.Memory.dll"
    },
    {
      "hash": "sha256-YQg8IghMY8pX7hWuiYAwk5ZUC0AMH9ocXIpoR0v5Pnw=",
      "url": "_framework\/System.Net.Http.dll"
    },
    {
      "hash": "sha256-wMwpwqN\/4NoGNubv9OAPyQ5wHGDqBfVwyXU2GnrGxqw=",
      "url": "_framework\/System.Net.Http.Json.dll"
    },
    {
      "hash": "sha256-XhaOHBnuuBTlmq9Ql4BNsyJMofu2PGm1jfMsb5SzJFE=",
      "url": "_framework\/System.Net.Primitives.dll"
    },
    {
      "hash": "sha256-YgI1h1OAlRlFjYLxihDndQElNJJQPfSqjlwZ\/Se\/DP8=",
      "url": "_framework\/System.Private.CoreLib.dll"
    },
    {
      "hash": "sha256-xSSJ+9XXUxenECnwe8bjHecAqYqZ+JPHthO9FXwOj7Y=",
      "url": "_framework\/System.Private.Uri.dll"
    },
    {
      "hash": "sha256-P11SLO9jG6JCbfAAsZqym+ASgvyFhaPQdkrW3sqQsJ8=",
      "url": "_framework\/System.Runtime.dll"
    },
    {
      "hash": "sha256-JI4+guYHKj5fqYDpsIHyPDahxGL0HaBf6diAoZ7NftI=",
      "url": "_framework\/System.Runtime.InteropServices.JavaScript.dll"
    },
    {
      "hash": "sha256-d2KyUrvcwKoW1p2fFbn+9S8MAFFWpDvA\/McxgmUYZq0=",
      "url": "_framework\/System.Text.Encodings.Web.dll"
    },
    {
      "hash": "sha256-G3mqwxP7plT8QPbjMqz87LfxB3tIrwVAi8iOMYG5xAA=",
      "url": "_framework\/System.Text.Json.dll"
    },
    {
      "hash": "sha256-hTtBBebB8GK8lA\/8LsOBK\/CLkHvOrPL5vXpZRxMUqmA=",
      "url": "sample2.styles.css"
    },
    {
      "hash": "sha256-uhotZszkBLq\/V8xt8UtpU6lGHEIqbqLsFUVGyelV2TU=",
      "url": "css\/app.css"
    },
    {
      "hash": "sha256-z8OR40MowJ8GgK6P89Y+hiJK5+cclzFHzLhFQLL92bg=",
      "url": "css\/bootstrap\/bootstrap.min.css"
    },
    {
      "hash": "sha256-gBwg2tmA0Ci2u54gMF1jNCVku6vznarkLS6D76htNNQ=",
      "url": "css\/bootstrap\/bootstrap.min.css.map"
    },
    {
      "hash": "sha256-jA4J4h\/k76zVxbFKEaWwFKJccmO0voOQ1DbUW+5YNlI=",
      "url": "css\/open-iconic\/FONT-LICENSE"
    },
    {
      "hash": "sha256-BJ\/G+e+y7bQdrYkS2RBTyNfBHpA9IuGaPmf9htub5MQ=",
      "url": "css\/open-iconic\/font\/css\/open-iconic-bootstrap.min.css"
    },
    {
      "hash": "sha256-OK3poGPgzKI2NzNgP07XMbJa3Dz6USoUh\/chSkSvQpc=",
      "url": "css\/open-iconic\/font\/fonts\/open-iconic.eot"
    },
    {
      "hash": "sha256-sDUtuZAEzWZyv\/U1xl\/9D3ehyU69JE+FvAcu5HQ+\/a0=",
      "url": "css\/open-iconic\/font\/fonts\/open-iconic.otf"
    },
    {
      "hash": "sha256-+P1oQ5jPzOVJGC52E1oxGXIXxxCyMlqy6A9cNxGYzVk=",
      "url": "css\/open-iconic\/font\/fonts\/open-iconic.svg"
    },
    {
      "hash": "sha256-p+RP8CV3vRK1YbIkNzq3rPo1jyETPnR07ULb+HVYL8w=",
      "url": "css\/open-iconic\/font\/fonts\/open-iconic.ttf"
    },
    {
      "hash": "sha256-cZPqVlRJfSNW0KaQ4+UPOXZ\/v\/QzXlejRDwUNdZIofI=",
      "url": "css\/open-iconic\/font\/fonts\/open-iconic.woff"
    },
    {
      "hash": "sha256-aF5g\/izareSj02F3MPSoTGNbcMBl9nmZKDe04zjU\/ss=",
      "url": "css\/open-iconic\/ICON-LICENSE"
    },
    {
      "hash": "sha256-waukoLqsiIAw\/nXpsKkTHnhImmcPijcBEd2lzqzJNN0=",
      "url": "css\/open-iconic\/README.md"
    },
    {
      "hash": "sha256-4mWsDy3aHl36ZbGt8zByK7Pvd4kRUoNgTYzRnwmPHwg=",
      "url": "favicon.png"
    },
    {
      "hash": "sha256-DbpQaq68ZSb5IoPosBErM1QWBfsbTxpJqhU0REi6wP4=",
      "url": "icon-192.png"
    },
    {
      "hash": "sha256-oEo6d+KqX5fjxTiZk\/w9NB3Mi0+ycS5yLwCKwr4IkbA=",
      "url": "icon-512.png"
    },
    {
      "hash": "sha256-Dg2uj5FTONMutQFA\/bavi4qjP\/n7cs5KSlHxzSeUlvU=",
      "url": "index.html"
    },
    {
      "hash": "sha256-ktQ0GNo4y\/NR+Jokrj566okRN7YiZQ\/64bW4b4Oyg9Y=",
      "url": "manifest.json"
    },
    {
      "hash": "sha256-enKgCMkYmCpfEcmg6Annbmc40VZ\/A6aYYSQjZfVn2cU=",
      "url": "sample-data\/weather.json"
    }
  ],
  "version": "fRK4QZQj"
};
```

## Fix

前面講了那麼多就是為了要搞懂原理，接下來就是修正問題，`service-worker-assets.js` 是編譯才產生，因此需要想辦法讓我們新增的檔案能夠被加入，在[微軟的文件](https://learn.microsoft.com/en-us/aspnet/core/blazor/progressive-web-app?view=aspnetcore-7.0&tabs=visual-studio#control-asset-caching)中其實有提到，如何增加靜態檔案的快取。
在 `csproj` 中設定 `ServiceWorkerAssetsManifestItem` 檔案以及輸出的路徑。

```xml
  <ItemGroup>
    <ServiceWorkerAssetsManifestItem Include="wwwroot\113年辦公日曆表.pdf"
                                     RelativePath="wwwroot\113年辦公日曆表.pdf"
                                     AssetUrl="wwwroot\113年辦公日曆表.pdf" />
  </ItemGroup>
```

設定後，執行 `publish` 的指令。

```shell
# 習慣先清除，避免有不預期的檔案影響
rm -rf .\Release
dotnet publish -c Release -o Release
```

執行指令不用急著去把系統跑起來，可以先看看 `service-worker-assets.js` 有沒有如預期地把檔案加進去，使用搜尋的方式找 `pdf` 字串，就能找到檔案如預期的被加入了。

```javascript
self.assetsManifest = {
  "assets": [
    {
      "hash": "sha256-uXu0r9IYHvtbwnfPL5kHgiHaOTGk6IfnbpfJDHzN3ZU=",
      "url": "113年辦公日曆表.pdf"
    }
  ],
  "version": "Ku+lgBpy"
};
```

接著使用前面架網站的方式來啟動，用瀏覽器的方式來看是否可以正常下載；可以發現還是失敗，偵錯 `onFetch` 看到 `request.mode` 是 `navigate`，因此會去走一般的請求。

### 解法一

調整 `onFetch` 的邏輯，讓 pdf 副檔名可以走快取的邏輯。

```javascript
async function onFetch(event) {
    let cachedResponse = null;
    if (event.request.method === 'GET') {
        // 調整這邊的邏輯
        const shouldServeIndexHtml = event.request.mode === 'navigate' && !event.request.url.endsWith('pdf');

        const request = shouldServeIndexHtml ? 'index.html' : event.request;
        const cache = await caches.open(cacheName);
        cachedResponse = await cache.match(request);
    }

    return cachedResponse || fetch(event.request);
}
```

> 如果有多個檔案的話，建議用一個資料夾會比較好控制

### 解法二

前面的問題在 Dotnet 8 有被調整過，可以參考這個 [github issue](https://github.com/dotnet/aspnetcore/issues/40678)，基本上就是有在 assets 控制下的就會採用快取，底下是 Dotnet 8 部分修正的內容。

> 個人覺得這個解法才是正確的

```javascript
const base = "/";
const baseUrl = new URL(base, self.origin);
const manifestUrlList = self.assetsManifest.assets.map(asset => new URL(asset.url, baseUrl).href);

async function onFetch(event) {
    let cachedResponse = null;
    if (event.request.method === 'GET') {
        // For all navigation requests, try to serve index.html from cache,
        // unless that request is for an offline resource.
        // If you need some URLs to be server-rendered, edit the following check to exclude those URLs
        const shouldServeIndexHtml = event.request.mode === 'navigate'
            && !manifestUrlList.some(url => url === event.request.url);

        const request = shouldServeIndexHtml ? 'index.html' : event.request;
        const cache = await caches.open(cacheName);
        cachedResponse = await cache.match(request);
    }

    return cachedResponse || fetch(event.request);
}
```

### 解法三

這個解法應該是最簡單的，採用標準 html 的作法，也就是加上 `download` 的屬性。

> murmur: 雖然這是標準但就算沒有加也不應該去走路由阿

```html
<a href="113年辦公日曆表.pdf" download>台灣 2024 年行事曆</a>
```

## Reference

- [ASP.NET Core Blazor Progressive Web Application (PWA) | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/blazor/progressive-web-app?view=aspnetcore-7.0&tabs=visual-studio#control-asset-caching)
- [Static files in ASP.NET Core | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/static-files?view=aspnetcore-7.0&viewFallbackFrom=aspnetcore-2.1&tabs=aspnetcore2x)
- [Host ASP.NET Core on Linux with Nginx | Microsoft Learn](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-7.0&tabs=linux-ubuntu#use-a-reverse-proxy-server)
- [ServiceWorker - Web APIs | MDN (mozilla.org)](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorker)
- [<a>: The Anchor element - HTML: HyperText Markup Language | MDN (mozilla.org)](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a)