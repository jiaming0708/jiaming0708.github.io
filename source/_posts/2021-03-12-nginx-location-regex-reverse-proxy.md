---
title: '[nginx] 設定reverse proxy當location使用regex'
date: 2021-03-13 09:03:58
categories:
- nginx
tags:
- nginx
---

一個網站有部分頁面是需要連到其他網站的，但又希望網址改變，這時候我們可以使用 reverse proxy 的技術來達成，今天要來分享的是如何在 nginx 中完成這樣的需求。

> reverse proxy 可以參考我之前 {% post_link reserve-proxy 寫的這篇 %}

<!-- more -->

# proxy_pass

先來個基本題，使用者進到 **course** 這個路徑時，希望能夠開啟維持一樣的 url 但是內容呈現在 webflow 上做好的一頁，我們所要使用的指令叫做 `proxy_pass`。

```nginx
server {
  location ~ /course {
    proxy_pass https://mkt-hk.webflow.io/hkdse-courses;
  }
}
```

# proxy_pass 與 regex

進階題來了，想要限制只有特定前綴 **zh-hk** 以及 **en-hk** 的 course 路徑才顯示 academy 的內容。

先來列一下預期的 URL

* /zh-hk/course
* /en-hk/course

```nginx
server {
  location ~ /*-hk/course {
    proxy_pass https://mkt-hk.webflow.io/hkdse-courses;
  }
}
```

很開心的做完了，重起 nginx 卻發現到錯誤

> "proxy_pass" cannot have URI part in location given by regular expression, or inside named location, or inside "if" statement, or inside "limit_except"

當 location 是動態 (regex) 路徑時，就只能指定 domain 那層，後面的 folder 不能存在。雖然只有兩個前綴，可以簡單的重複寫兩次，但總是覺得這樣很笨。

查了相關的設定以後，需要透過 **rewrite** 的方式，先讓這個頁面先轉到目標 folder，但又不能真的執行 (break)，再來執行 proxy_pass 就可以達成目的。

```nginx
server {
  location ~ /*-tw/course {
    rewrite ^ /hkdse-courses break;
    proxy_pass https://mkt-hk.webflow.io;
  }
}
```

# 參考

* [stackoverflow](https://stackoverflow.com/questions/53353572/proxy-pass-cannot-have-uri-part-in-location-given-by-regular-expression)
* [nginx](http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#break)