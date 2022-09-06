---
title: '[Angular] 動態載入Component'
categories:
- Frontend
- Angular
tags:
- Angular
---

最近在搞一個功能，目標是可以重新設定route，看到了一個功能`resetConfig`覺得可以用
就開始測試，沒想到一直遇到鬼，就是過不去
> No component factory found for CreateComponent. Did you add it to @NgModule.entryComponents?

<!--more-->

表示系統一直不認識這個component
但是看來看去ngModule我都有放進去該component
``` typescript
@NgModule({
  imports: [
    AppRoutingModule,
    CoreModule
  ],
  declarations: [
    AppComponent,
    CreateComponent
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

後來請教了[Kevin Yang](https://www.facebook.com/CKNotepad/)
才發現原來是少放了一個地方`entryComponents`
完整的如下
``` typescript
@NgModule({
  imports: [
    AppRoutingModule,
    CoreModule
  ],
  declarations: [
    AppComponent,
    CreateComponent
  ],
  bootstrap: [AppComponent],
  entryComponents: [CreateComponent]
})
export class AppModule { }
```

那entryComponents到底有什麼不一樣，首先來看看[官方文件](https://angular.io/docs/ts/latest/cookbook/ngmodule-faq.html#!#q-entry-component-defined)

---
### When do I add components to entryComponents?
Most application developers won't need to add components to the entryComponents.

Angular adds certain components to entry components automatically. Components listed in @NgModule.bootstrap are added automatically. Components referenced in router configuration are added automatically. These two mechanisms account for almost all entry components.

If your app happens to bootstrap or dynamically load a component by type in some other manner, you must add it to entryComponents explicitly.

Although it's harmless to add components to this list, it's best to add only the components that are truly entry components. Don't include components that are referenced in the templates of other components.

---

也就是說，想要動態載入一個沒有被用到的Component就必須要放到entryComponents中
這樣系統才會認識他，不會當他是壞人( ￣ c￣)y▂ξ