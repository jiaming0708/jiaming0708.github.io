---
title: '[Angular] 下載image並顯示'
date: 2018-02-13 00:29:29
categories:
- Front-end
- Angular
tags:
- Angular
---

# 緣起

最近公司的產品有一個需求，是要讓使用者自行更換網頁的圖片，所以就有上傳下載圖片的需求，後端API是直接看上傳什麼格式就下載什麼內容，但是這樣的資料其實在angular中是需要加工才能使用的，來記錄一下作法，廢話不多說上code

<!--more-->

# Download

這種檔案的下載，後端吐回來其實是blob，所以必須要將httpclient指定類型

```javascript
this.http.get(url, { responseType: 'blob' });
```

收到blob的資料後，要轉換成圖片其實有兩種作法，我比較偏好地一種，大家可以根據自己需求來使用

## url

使用window的[URL.createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)這個api將檔案直接由瀏覽器暫存於記憶體中，只要使用一個短短的url就可以做使用，免去`漏漏長`的字串

> 限定於ie10+使用，萬惡的IE...

```javascript
    this.http.get(url, { responseType: 'blob' })
      .pipe(
      map(data => {
        // 避免沒檔案回傳卻還是產生url然後變成叉燒包
        if (data.size > 0) {
          const urlCreator = window.URL;
          return this.sanitizer.bypassSecurityTrustUrl(urlCreator.createObjectURL(data));
        } else {
          return undefined;
        }
      }));
```



## base64

這個傳統的作法其實也叫做[data uri schema](https://en.wikipedia.org/wiki/Data_URI_scheme)，就是內容會比較肥大一點，但好處就是瀏覽器都支援

```javascript
    this.http.get(url, { responseType: 'blob' })
      .pipe(
        mergeMap(data => Observable.create(obs => {
          if (data.size > 0) {
            const reader = new FileReader();
            const reader = new FileReader();
            reader.onload = function () {
              obs.next(reader.result);
            };
            reader.readAsDataURL(data);
          } else {
            obs.next(undefined);
          }
        })));
```

