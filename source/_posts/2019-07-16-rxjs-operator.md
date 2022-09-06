---
title: rxjs之operator使用介紹
date: 2019-07-16 22:23:04
categories:
- Frontend
- RxJS
tags:
- RxJS
---

很多人在使用rx的時候，往往遇到的瓶頸是operator的使用，因為實在太多可以使用，今天要來分享一下我曾經用過以及常用的operator是哪些

<!-- more -->

# operator

從一開始寫Rx到現在，全部使用的operator有差不多18個左右，其中只有五個是最常用的，其他都是遇到需求而使用

> 目前官方大概有100個左右

## of

將參數輸出成資料流，常用建構observable之一

```javascript
of(1, 2, 3).subscribe(console.log);
// 1
// 2
// 3
```

## from

跟`of`類似，個人蠻常用在將陣列拆解

```javascript
from([1, 2, 3]).subscribe(console.log);
// 1
// 2
// 3
```

## fromEvent

監聽element的事件最佳利器！！沒有之一XD

```javascript
fromEvent(window, 'scroll').subscribe(console.log);
```

## map

這個是最常用之一，只要有任何的物件轉換就一定會使用到，而且一段裡面可能會使用好幾次

```javascript
map(p => ({
		id: p.id,
		name: p.Name,
	})
)
```

## mergeMap

把merge和map的功能合一，兩個observable資料是可以並行進來輸出

## switchMap

功能與`mergeMap`類似，資料進來後會取消前面的資料重新要一次

## concatMap

跟`mergeMap`類似，但會等到前一個observable結束才會執行，通常使用於前面的API結束後才能執行後面的API

## filter

顧名思義，就是過濾資料用，例如重複的資料不想要，或是只想要拿到某些資料

```javascript
from([true, false, true]).pipe(
  filter(p => !!p)
)
```

## tap

基本上寫rx應該要讓每一個function都是pure的，減少side effect
但如果真的再中間需要寫入一些值的話就可以使用，另一個使用狀況就是debug啦

```javascript
from([true, false, true]).pipe(
  filter(p => !!p),
  tap(console.log),
)
```

## takeUntil

比較多使用在unsubscribe一個observable，在angular中就是**ngOnDestroy**事件中，在react就是**componentWillUnmount**

```javascript
fromEvent(window, 'scroll').pipe(
  takeUntil(this.destory$)
)

destory$ = new Subject();

ngOneDestory(){
  this.destory$.next();
  this.destory$.complete();
}
```

## toArray

前面有說到會用`switchMap`把陣列拆解開，但最後的資料就是需要陣列，這時候就很適合使用

```javascript
of(1, 2, 3).pipe(
	toArray()
)
```

## reduce

資料整理上的好幫手，可以把陣列拆解，然後可以輸出任何類型的資料

```
from([1, 2, 3]).pipe(
	reduce((acc, value) => {
		acc.push(value * 2);
	}, [])
)
```

## catchError

我主要應用在串接API，當發生錯誤時要先做處理然後再讓呼叫端拿到error

```javascript
of(undefined).pipe(
	map(p => p.name),
  catchError(p => {
    // do something
    // 結束整個observable
    return Observable.throw(p);
  })
)
```

## debounceTime

收到資料沒有變化多久的時間後就會觸發，適合在不想要被頻繁觸發的事件上，例如`autocomplete`

```javascript
fromEvent(input, 'keyDown').pipe(
	debounceTime(200)
)
```

## delay

跟sleep差不多的概念（笑），基本上只使用過一次，demo project為了模擬api的效果才用的

## bufferCount

累積n筆資料以後才輸出，輸出為陣列，但要注意，如果數量不夠，那些資料就會一直卡在拿不到！基本上也只使用過一次

```javascript
of(1, 2, 3).pipe(
	bufferCount(2)
)
```

## pairwise

情境是為了要可以拿到連續配對的資料 `[1, 2] [2, 3] [3, 4]`；只使用過一次

> 補充，也能用在scroll事件，可以用來判斷前次和這次的捲軸位置

```javascript
of(1, 2, 3).pipe(
	pairwise()
)
```

## bufferTime

資料需要被累積一定時間後再一起輸出，輸出為陣列；只使用過一次

```javascript
of(1, 2, 3).pipe(
	bufferTime(100),
)
```

## 小結

好的，介紹完曾經使用過的operator，有沒有發現有不少都非常的難用到，因為情境太少，畢竟大部分的情況都還是在整理資料或是串接的用途

# subject

為什麼要額外提subject呢，因為這個功能實在太好用啦，使用情境來列一下

* 變數值是一樣的，但希望被重複觸發

  > 例如alert message，什麼時候不會管，外層只需要呼叫讓component顯示就好

```javascript
//只會觸發一次
var openDialog = true;
//<app-dialog [open]="openDialog" />

var openDialog$ = new Subject();
openDialog$.next(true);
//<app-dialog [open]="openDialog$" />
```

* 手動控制資料時間點，例如destory
* 需要有預設值或是取得前幾次的結果（其他subject）

# 結論

operator常用的就那幾個，剩下都是屬於情境型的應用，再沒有看到那種case的情況下，就是不知道這些要怎麼使用，所以也不用擔心，先開始用並且寫下去就知道
我也寫過一個map露露等5.60行的，先交差，後面再來想要怎麼改善，一次到位永遠都是最痛苦的

看完這篇也能看看另一篇 {% post_link rx-sample %} 會更有感覺喔!

> 如果覺得挑戰不夠，可以看看這個[repo](https://github.com/jiaming0708/draw-demo)，絕對給你滿滿的應用！