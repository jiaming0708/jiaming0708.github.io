---
title: 如何再Wallaby取得package.json檔案
date: 2018-05-28 20:21:12
categories:
- Frontend
- Angular
tags:
- Angular
- Wallaby
---

最近看到 Kevin 的一篇[文章](https://www.facebook.com/CKNotepad/posts/542075559526993)，說到有用到alias就必須在wallaby另外設定，想說很久沒有弄前陣子上線的專案，開啟來設定看看，沒想到設定後完全沒有用

<!--more-->

後來檢查發現到，竟然是因為當初為了要拿到`package.json`的version而在`environment`加上require直接取值，像下面這樣

```typescript
export const environment = {
  production: false,
  appVersion: require('../../package.json').version
};
```

因為wallaby根本不認識`package.json`的檔案，因此我們必須要先讓wallaby認識，加上兩個地方的設定

```javascript
module.exports = function (wallaby) {
    return {
        files: [
            'package.json'
            //...
        ],
        middleware: function (app, express) {
            var path = require('path');
            app.use('/package.json', express.static(path.join(__dirname, 'package.json')));
            //...
        }
    }
}
```

這樣就可以打完收工，繼續使用`wallaby`啦

> 附帶一提，如果有用到`chart.js`，也一樣要寫進`files`