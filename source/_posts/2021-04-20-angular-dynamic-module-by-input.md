---
title: '[Angular]透過輸入的方法動態載入模組'
date: 2021-04-20 14:30:50
categories:
- Frontend
- Angular
tags:
- Angular
---

在Angular中，除了先在Route定義好路徑並且載入模組以外，也能透過動態的方式去載入，能夠讓程式更加的靈活。今天想要試著用輸入的方法來做到動態載入模組，最後結果是失敗的，要來看看編譯後的結果，試著去理解原因。

<!-- more -->

動態載入模組可以參考以下的文章

* [[Angular 大師之路] Angular 8 之後動態載入模組的方法 (非延遲載入路由)](https://wellwind.idv.tw/blog/2019/06/04/angular-8-dynamic-load-module/)
* [[Angular] 手動創造出 Lazy Loading 的效果](https://blog.kevinyang.net/2017/11/08/manual-lazy-loading/)

# 載入模組固定

## 編譯前

```typescript
onClickLazyModule() {
  import('./lazy/lazy.module').then(loadedModule => {
    console.log('123', loadedModule);
  }); 
}
```

## 編譯後

```javascript
onModuleChange(path) {
    __webpack_require__.e(/*! import() | lazy-lazy-module */ "lazy-lazy-module").then(__webpack_require__.bind(null, /*! ./lazy/lazy.module */ "g5p6")).then(loadedModule => {
        console.log('123', loadedModule);
    });
}
```

# 載入模組不固定

模組路徑透過輸入的方法載入

## 編譯前

```typescript
onClickLazyModule(pathModule) {
  import(pathModule).then(loadedModule => {
    console.log('123', loadedModule);
  }); 
}
```

## 編譯後

```javascript
/***/ "+Huz":
/*!***************************************!*\
  !*** ./src/app lazy namespace object ***!
  \***************************************/
/*! no static exports found */
/***/ (function(module, exports) {

function webpackEmptyAsyncContext(req) {
	// Here Promise.resolve().then() is used instead of new Promise() to prevent
	// uncaught exception popping up in devtools
	return Promise.resolve().then(function() {
		var e = new Error("Cannot find module '" + req + "'");
		e.code = 'MODULE_NOT_FOUND';
		throw e;
	});
}
webpackEmptyAsyncContext.keys = function() { return []; };
webpackEmptyAsyncContext.resolve = webpackEmptyAsyncContext;
module.exports = webpackEmptyAsyncContext;
webpackEmptyAsyncContext.id = "+Huz";
  
onModuleChange(path) {
    __webpack_require__("+Huz")(path).then(loadedModule => {
        console.log('123', loadedModule);
    });
}
```

## 編譯警告

```shell
Warning: ./src/app/app.component.ts 14:8-20
Critical dependency: the request of a dependency is an expression
    at ImportContextDependency.getWarnings (/Work/test/node_modules/webpack/lib/dependencies/ContextDependency.js:40:18)
    at Compilation.reportDependencyErrorsAndWarnings (/Work/test/node_modules/webpack/lib/Compilation.js:1454:24)
    at /Work/test/node_modules/webpack/lib/Compilation.js:1258:10
    at _next0 (eval at create (/Work/test/node_modules/tapable/lib/HookCodeFactory.js:33:10), <anonymous>:30:1)
    at eval (eval at create (/Work/test/node_modules/tapable/lib/HookCodeFactory.js:33:10), <anonymous>:43:1)
    at runMicrotasks (<anonymous>)
    at processTicksAndRejections (internal/process/task_queues.js:97:5)
 @ ./src/app/app.module.ts
 @ ./src/main.ts
 @ multi ./src/main.ts
```

# 結論

從上面的結果來看，可以看到當使用 import 沒有給路徑的話，在編譯的時候就會直接給警告，並且輸出的結果永遠都是 `Cannot find module`。

要這麼動態的載入，在angular的編譯機制中，就會出現這樣的結果，也許要workaround才能做到，那這邊我就沒有深入去研究了，如果你有其他作法，歡迎留言給我。

