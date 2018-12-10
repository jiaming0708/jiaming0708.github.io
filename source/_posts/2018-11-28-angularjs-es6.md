---
title: angularjs使用es6開發-grunt+webpack+babel
date: 2018-11-28 09:35:17
categories:
- Front-end
- AngularJS
tags:
- AngularJS
---

在前一篇{% post_link ng-admin %}中，有提到要使用es6來撰寫，但是angularJS預設是es5，讓我們看一下該怎麼進行整合，使用更強更好用的es6來快樂的開發angularJS

<!--more-->

# 開始之前

要開始整合編譯工具之前，應該要先知道架構面的事情，原本在寫angularJS時，每個controller或是directive還是會拆分檔案來進行開發，像是下面這樣

-src
--directive.js
--directive2.js
--controller.js
--controller2.js
-index.html

上線到server的時候，會將所有的js變成一個檔案，並且進行ugly，大概會是像這樣

-disc.js
-index.html

所以在整合es6的時候，我們就要先改變一下寫法，將directive、controller等寫成一個標準export的js檔案，像是下面的寫法

```javascript
// duplicate.js
function duplicate (Restangular) {
  return {
    restrict: 'E',
    scope: {
    },
    template: '<span class="fa fa-clone" aria-hidden="true">Duplicate post</span>',
    link: function (scope, element, attrs) {
        //do something
    },
  };
}

duplicate.$inject = [
  'Restangular',
];

export default duplicate;

```

```javascript
// app.js
import duplicate from './duplicate';

angular.module('app')
.direcitive('duplicate', duplicate);
```

這時候呢，我們只要針對`app.js`這個檔案做編譯，其他被引用到的就會被編譯到同一個檔案中，那之前所用到把各個檔案集中到一個的作法也就不用

# 編譯工具

現在有兩個方法可以將es6編譯成es5，接下來就來詳細的介紹一下這些工具

## typescript

typescript是一個超集合，所以可以單純的用ts的檔案但內容全部是一般js的寫法，在angular裡面就開始全部使用ts，所以採用這個也是一個很好的選擇

> 要注意一下，typescript本身是一個語言，支援js所以有編譯的功能，但絕對比單純的編譯器還強

## babel

這套算是目前最多人使用的吧，基本上使用react的人幾乎都是用這個工具在進行編譯，官方也提供讓你在[線上](https://babeljs.io/)可以觀看輸出結果，現在babel的版本已經來到7，也跟之前有一些不一樣，所以要注意一下

# 整合

在這次的整合中，我是使用 `grunt` + `webpack` + `babel`作為最後的選擇，先說一下之前嘗試的歷程

> 因為很少這樣整合，所以在測試上花了很多時間orz

* typescript
  在angular中使用過ts，當然想到的第一個整合會是這個，但是在整合的時候發現輸出的結果跟預期的不同，例如`import`一個檔案進去，但還是有require之類的存在，導致在處理上更加麻煩

  > 建議可以的話還是使用ts，應該會是最佳選擇

* babel
  使用babel本身是非常正常的，但是一跟`grunt`會編譯失敗，一直找不到真正的原因，時間上的問題，所以只好放棄

  > "grunt": "^1.0.3"
  > "grunt-cli": "^1.2.0" 

* webpack
  大家最常見的方法，使用`webpack`+`babel`的設定，然後再跟`grunt`整合，這邊來看一下怎麼操作

> "webpack": "^4.23.1"
> "@babel/core": "^7.1.2"

```javascript
//將webpack的設定寫在另一個檔案中
const webpackConfig = require('./webpack.config.js');

grunt.initConfig({
	webpack: {
	  options: {
	    stats: true,
	  },
	  prod: webpackConfig,
      //預設mode是prod，所以在開發時就不用特別給
	  dev: Object.assign({ mode: 'none' }, webpackConfig),
	},
});
```

# 結論

建議有在繼續寫angularJS的都應該要使用編譯工具，讓開發上更加輕鬆（因為使用angularJS已經夠辛苦），時間除了花在找工具外，也有一部分是在把原本寫不好的code修好，這種整合的期間很痛苦，但弄好的時候真的很爽，都可以使用es6的語法，輕鬆寫code