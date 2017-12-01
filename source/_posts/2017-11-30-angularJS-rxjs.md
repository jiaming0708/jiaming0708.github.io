---
title: AgnularJS與RxJS結合-監聽model變動
date: 2017-11-30 21:26:02
categories:
- Front-end
- AgnularJS
tags:
- AngularJS
---

# 前情提要

前陣子在公司的舊有專案有個需求，是要做AutoComplete的效果，那時候第一時間腦袋就是浮現了RxJS，用其他作法不是不行，只是剛好學了Rx就是想要用看看（其實想炫技～大誤）

> angularJS: 1.5.5
> RxJS: 5.5.2

# AngularJS+RxJS

## Rx.Angular.JS

決定好要使用以後先搜尋看看這相關的資料，沒想到竟然有整合好的套件[`rx.angular.js`](https://github.com/Reactive-Extensions/rx.angular.js/)，那當然就是馬上npm來用用看啦
但這個套件其實已經很久沒維護，RxJS也沒有更新，所以奉勸想要用的朋友請三思！

## RxJS

還是直接安裝原生的最快（這邊不談論bundle大小的問題...），在使用的頁面中直接import就可以使用!

> npm install rxjs

# ngModel+RxJS

RxJS可以用事件進行監聽，這作法會類似於原生js的寫法，但在angularJS的世界中，因為是用ngModel進行資料的雙向繫結，在這件事情上面就會比較麻煩一點

## 建立一個監聽器

使用 `$watch` 的方式做監聽model的變化，並且將資料透過Observer發送出去

```javascript
import Rx from 'rxjs/Rx'

Rx.Observable.$watch = (scope, watchExpression, objectEquality) => {
    return Rx.Observable.create(observer => {
        // 當model變動時，送資料給訂閱者
        function listener(newValue, oldValue) {
            observer.next({ oldValue: oldValue, newValue: newValue })
        }

        // 使用$watch的方式監聽model的變化
        return scope.$watch(watchExpression, listener, objectEquality)
    })
}
```

## 開始監聽

```javascript
Rx.Observable.$watch($scope, () => this.query.Keyword)
    .debounceTime(200)
    .subscribe(p => console.log(p));
```

透過這個作法，只要開始輸入資料就會開始接收到資料，這時候在來搭配後端的api去做查詢的動作，基本上AutoComplete的功能就算是完成

# 結論

AngularJS雖然沒有像Angular一樣完美的結合了RxJS，但要使用上還是沒有問題，而且開始用以後就會停不下來，目前公司的產品已經開始被我慢慢使用，尤其是要多個API串接，然後有順序性那種，使用起來根本就是爽！