---
title: '[Angular] DI注入環境變數'
date: 2017-03-11 16:01:20
updated: 2017-03-11 16:01:20
categories:
- Frontend
- Angular
tags:
- Angular
---

話說最近弄了一個題目練習Angular，搭配cli使用(只有方便而已啦!!我以前是在白痴什麼orz...)
自己想了一個需求，就是我的API Server有分prodction和development

<!--more-->

## 基礎應用
首先衍生出來的就是要設定在哪，然後怎麼用
cli中有一個environment的資料夾，底下兩個檔案，`environment.ts、environment.prod.ts`
很簡單的就是在裡面增加一個屬性叫做apiServer，並且應用到service中

![file path](filePath.png)

``` typescript
export const environment = {
  production: false,
  apiServer: 'https://192.168.100.1'
};

import { environment } from "../../environments/environment";
@Injectable()
export class BeverageService {
  getBeverageList(): Observable<BeverageData[]> {
    return this._http.get(environment.apiServer+ "Beverage")
      .map(response => response.json() as BeverageData[])
      .catch(this.handleError);
  }
}
```

## 進階應用
這樣寫其實已經滿足我們需求，但是有點low
來設想一個情境，如果之後我們的apiServer名字改掉的話，是不是變成service裡面所有使用到environment.apiServer這個全部要換掉
其實angular本身有提供一個東西，叫做[DI](https://angular.io/docs/ts/latest/cookbook/dependency-injection.html#!#usevalue)
那我們可以把他做一個轉換，這樣在使用上，就可以脫離environment的直接影響
只要在providers放進去，並且在constructor取得就好

```typescript
import { environment } from "../../environments/environment";
@NgModule({
  providers: [
    { provide: 'apiServer', useValue: environment.apiServer }
  ]
})

@Injectable()
export class BeverageService {
  constructor(private _http: Http, @Inject('apiServer') private apiServer: string) {
    this.onInit();
  }
}
```

## DI學習參考
[oomusou的文章，深入探討 Angular 的 DI 與 Provider](https://old-oomusou.goodjack.tw/angular/di/)
