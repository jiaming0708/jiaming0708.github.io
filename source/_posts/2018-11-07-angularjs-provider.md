---
title: '[AngularJS] Provider探究'
date: 2018-11-07 12:44:39
updated: 2018-11-07 12:44:39
categories:
- Frontend
- AngularJS
tags:
- AngularJS
---

# 介紹

> 此篇是AngularJS 1.x，不是2+以上的喔！

在AngularJS中提供一個很好用的機制叫做DI(Dependecy Injection 依賴注入)，這是一個design pattern，可以降低測試及宣告的程度，想要了解更完整的觀念可以參考一下[燈哥的文章](https://old-oomusou.goodjack.tw/angular/di/)，這邊就不探討。

<!--more-->

官方也專門寫了一篇來介紹整個的機制，但其實真的很容易搞混，所以特別寫了這篇文章來記錄。
首先先來看看有哪些是可以被inject的

- provider
- constant
- value
- factory
- service
- decorator
- filter

# DI時機

這麼多的類型，他們可以被inject的時機也會有所不同，可以參考[官方文件](https://docs.angularjs.org/guide/di#using-dependency-injection)，在使用上要注意一下要寫的service是屬於什麼層級

| 區塊     | 限制                                                         |
| -------- | ------------------------------------------------------------ |
| run      | 除了provider以外的都可以用                                   |
| config   | 除了provider、constant以外的都不能用                         |
| provider | 除了provider、constant以外的都不能用，如果inject其他provider要注意**順序** |

# Provider

那到底什麼是`provider`，為什麼會有這麼多限制在，其實前面所提到的類型基本上都是透過provider來實作出來的，我們先來看一下怎麼寫一個provider

```javascript
//provider
angular.module('app')
.provider('first', function(){
    function console (str) {
        console.log(str);
    };
    // $get一定要有的
    this.$get = function(){
    	return {
            console: console,
    	}
    };
})
//config
.config(['firstProvider', function(firstProvider){
    //呼叫getData
    firstProvider.$get().console('config');
}])
//directive
.directive('myDirective', ['first', function(first){
    first.console('directive');
}]);
```

可以發現到`firstProvider`在兩個地方是完全不一樣的表現

* 在大部分的使用情況下inject都會直接拿到`$get`的function內容
* 只能使用provider的地方，一定要在使用時必須要像這樣`{name}Provider`，而且只能自己呼叫 `provider.$get()`去拿到裡面的內容

也就是說今天我們有一個provider只打算提供給`config`、`provider`使用的話，可以直接寫在provider的object中，但`$get`還是必須要有的，但我們可以不回傳任何東西

```javascript
//provider
angular.module('app')
.provider('first', function(){
    this.console = function (str) {
        console.log(str);
    };
    // $get一定要有的
    this.$get = function(){
    };
});
```

## 實作類型

`constant`、`value`、`factory`、`service`都是透過provider來實作的，那實際上到底是怎麼運作的呢，我們可以來看一下這段在[官方的範例](https://docs.angularjs.org/guide/module#module-loading)

```javascript
angular.module('myModule', []).
  value('a', 123).
  factory('a', function() { return 123; }).

// is same as

angular.module('myModule', []).
  config(function($provide) {
    $provide.value('a', 123);
    $provide.factory('a', function() { return 123; });
  });
```

可以發現到其實都是透過`$provider`來做註冊，所以我們當然也能夠透過provider來完成實作，各個的詳細範例可以看一下[官方文件](https://docs.angularjs.org/guide/providers)

# 結論

講了那麼多，主要就是要知道`provider`的使用上一些要注意的點，當然你一定會想說既然provider這麼強大，基本上都可以用，那為什麼我們不要全部都寫provider就好呢？
其實當然可以，但是寫起來會比較麻煩一點阿，沒事還要自己寫那些`$get`的內容，不如直接用已經提供的服務不是更好！
