---
title: '[Angular] 2.0.0RC5升級2.4.0的注意事項'
date: 2017-02-19 19:25:08
categories:
- Frontend
- Angular
tags:
- Angular
---

### 前言
之前自己玩的一個小專案，是一路從beta開始開發一直到RC5
後來就都沒有變，一直到把功能開發完上線

<!--more-->

最近參加線上讀書會，討論到一個算是bug的功能，kevin就說趕快更新到最新吧!
> *change事件，裡面取得到ngModel的參數值是舊的，導致要傳入$event進去

### 正文
來聊聊升級的坑吧，原本以為是把package.json改一改就好，現實總是殘酷的!

1.component中移除pieps、directives兩個屬性，改成在@NgModule中宣告

2.所有使用到的pipe、directive、component都需先在@NgModule中宣告

3.route寫法改變，一律採用class方式宣告，分檔時先使用NgModule載入

``` typescript
//舊的寫法
export const routing = RouterModule.forRoot(appRoutes, { useHash: true });
export const appRoutingProviders: any[] = [];
@NgModule({
  providers: [appRoutingProviders],
})
//新的寫法
@NgModule({
  imports: [
    RouterModule.forRoot(appRoutes, { useHash: true })
  ],
  exports: [
    RouterModule
  ]
})
export class AppRoutingModule { }

```