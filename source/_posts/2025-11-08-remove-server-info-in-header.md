---
title: 移除 Nginx/IIS 伺服器資訊
date: 2025-11-08 09:47:18
updated: 2025-11-08 09:47:18
categories:
- Infra
tags:
- nginx
- IIS
thumbnail:
---

公司最近做了資安檢測，有被檢測出來 response 中不應該帶入 server 的版本相關資訊，才不會因此而被拿來做為攻擊的依據，以下會說明 IIS 和 nginx 各自怎麼設定。

<!-- more -->

## IIS

如果網站數量比較少，可以考慮用 web.config 來設定，因為公司有多台主機而且服務眾多，以下選擇從 IIS 上每台作設定。

> 使用 web.config 的設定可以參考保哥的文章 [如何設定 ASP.NET Core 在發行到 IIS 時移除 X-Powered-By 標頭](https://blog.miniasp.com/post/2023/10/27/Remove-HTTP-X-Powered-By-Header-from-IIS-and-ASP-NET-Core)

在 IIS root 的右側找到 `管理` -> `設定編輯器`

![iis-root](iis-root.png)

點開後總共要改三個東西，修改後要將 IIS 重啟

### Server Header

在上面的區段找到 `system.webServer/security/requestFilter`，在下方的內容找到 `revmoeServerHeader` 將值改為 `true`，修改後要點選右方 **動作** 的 **套用**
![iis_server_header](iis_server_header.png)

### X-Powered-By

在區段找到 `system.webServer/httpProtocol`，下方的 `customHeader` 點開來

![iis_x_powered_by1](iis_x_powered_by1.png)

先點選 `X-Powered_By`項目，再點選右邊的移除，修改後要點選右方 **動作** 的 **套用**
![iis_x_powered_by2](iis_x_powered_by2.png)

### Version Header

找到區段 `system.web/httpRuntime`，選擇 `enableVersionHeader` 將值改為 `false`，修改後要點選右方 **動作** 的 **套用**

![iis_version_header](iis_version_header.png)

## Nginx

打開 `nginx.conf` 找到 `http` 的區塊作以下的調整

### Hide Version

只要加上 `server_tokens` 的設定，就可以版本隱藏掉

```
http {
   ...
   server_tokens off;
   ...
}
```

### Clear Server Header

想要把 nginx 的資訊完全隱藏掉的話，可以安裝模組


```sh
apt install nginx-extras
```


```
http {
   ...
   more_clear_headers Server;
   ...
}
```



組合起來就是這樣

```
http {
   ...
   server_tokens off;
   more_clear_headers Server;
   ...
}
```



