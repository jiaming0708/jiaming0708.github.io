---
title: quokka搭配webpack設定
date: 2019-02-15 22:44:03
updated: 2019-02-15 22:44:03
categories:
- Frontend
- Testing
tags:
- Quokka
- Testing
---

[quokka](https://quokkajs.com/)是一個測試工具，支援蠻多的編輯器，可以直接即時的就看到結果，所以大部分都是開發或是測試的時候可以拿出來使用

在使用webpack作為打包編譯的工具的時候，有一些設定會讓quokka看不懂，這邊就來介紹一下怎麼設定

<!-- more -->

# alias

在專案中通常會加上alias的設定，這樣可以讓import比較乾淨，大概會像是這樣設定

```javascript
module.exports = () => {
    return {
        //...
        resolve: {
            alias: {
                '@app': resolve(__dirname, './app')
            }
        },
    };
}
```

那在程式中通常會這樣寫

```javascript
import api from '@app/services/api';
```

但是這個時候使用quokka會告訴你找不到這個module，根據[官方的文件](https://quokkajs.com/docs/configuration.html)可以透過設定`.quokka`的檔案(*json  format*)，來讓工具看的懂

## without babel

alias有兩種方法，一個是有babel，當然另一種就是沒有，這兩個都有寫好的套件可以使用
首先介紹一下沒有babel的情況下要使用[alias-quokka-plugin](https://github.com/Gozala/alias-quokka-plugin)

```shell
yarn add --save --dev alias-quokka-plugin
```

再來就是把`.quokka`的檔案產生好

```json
{
  "plugins": ["alias-quokka-plugin"],
  "alias": {
      "@app": "./app",
  }
}
```

設定完以後就不用怕啦，開始使用在程式裡面吧

```javascript
const api = require('@app/services/api');
```

## with babel

有使用babel的話，我們要安裝另一個套件[babel-alias-quokka-plugin](https://github.com/timhuff/babel-alias-quokka-plugin)

```shell
yarn add --save --dev babel-alias-quokka-plugin
```

改一下`.quokka`檔案的內容

```json
{
    "babel": true,
    "plugins": [
      "babel-alias-quokka-plugin"
    ],
    "alias":{
      "@app":"./app"
    }
  }
```

可以看到這邊主要多了一個設定叫做`babel`，那預設會載入`.babelrc`並且env是**test**，當然你也可以特別指定env之類的，但我是全部都交給`babelrc`就好，`.quokka`只做最基本的設定

# global variable

很開心的設定完上面的步驟，一按下去執行，🤬又有錯誤，這次是**`__CLIENT__`**找不到，突然想到在webpack中有定義了一些屬於全域型的變數

```javascript
// in webpack.config.js
new webpack.DefinePlugin({
    __CLIENT: true,
});
```

## 自定plugin

這時候我們就要採用自訂plugin的方式，宣告一個變數讓quokka看懂，這時候就先產生一個`quokkaPlugin.js`的檔案

```javascript
global.__CLIENT__ = true;
```

在`.quokka`的設定中，把這個檔案當做plugin放進去

```json
{
    "babel": true,
    "plugins": [
      "./quokkaPlugin.js",
      "babel-alias-quokka-plugin"
    ],
    "alias":{
      "@app":"./app"
    }
  }
```

# 結論

終於設定完畢，可以正常的使用`quokka`這個工具，如果你的import很多的話，可能會使用到**Pro**的版本，可以先看一下官方的差異介紹，試用後再決定要不要買

如果你是有套用完整的測試框架像是jest或是protractor，就要改使用[wallaby](https://wallabyjs.com/)，絕對可以讓你享受到TDD的紅綠燈開發方式

再設定上面如果有一些問題的話可以去參考官方的[repo](https://github.com/wallabyjs)，裡面有非常多種的範例