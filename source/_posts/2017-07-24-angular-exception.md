---
title: '[Angular/AngularJS] Exception Handle'
date: 2017-07-24 22:00:20
updated: 2017-07-24 22:00:20
categories:
- Frontend
- Angular
tags:
- Angular
- AngularJS
---

今天來看一下Angular和AngularJS如何去handle在網站中任一個地方發生的exception
想要把這種資料回寫到後端，我們可以怎麼去實作他

<!--more-->

# Angular作法
在Angular可以攔截在全域未handle到的exception，可以看一下[官方文件](https://angular.io/api/core/ErrorHandler)

只要照著寫一個class `implements ErrorHandler`就可以
最簡單的錯誤就是在呼叫api的時候沒有去handle exception(想辦法讓api呼叫失敗)
這時候就會進入到自己寫的那個class中

``` typescript
import { ErrorHandler } from "@angular/core";

export class MyErrorHandler implements ErrorHandler {
    handleError(error) {
        //call backend api
        console.log(`this is my error handle, ${error}`);
    }
}
```

# AngularJS作法
因為工作上還有用到AngularJS，同事就問到有沒有這種東西
於是查詢了一下[官方文件](https://docs.angularjs.org/api/ng/service/$exceptionHandler)

作法跟Angular蠻像的，果然是前後代XD

同樣的照著官方文件做，寫一個factory叫做`$exceptionHandler`，overwrite原生的handler
一樣可以輕鬆的搞定需求
``` javascript
angular.module('exceptionModule')
    .factory('$exceptionHandler', ['$log', function ($log) {
        return function myExceptionHandler(exception, cause) {
            //call Backend api
            $log.error(`my error handler, ${exception}`, cause)
        }
    }]);
```