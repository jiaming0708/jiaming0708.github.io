---
title: 使用AngularCli搭配ng-packagr建立元件
date: 2018-06-13 20:09:31
categories:
- Front-end
- Angular
tags:
- Angular
---

平常我們都用Angular Cli來建立專案，但如果有需要製作元件給別人使用，這時候應該怎麼做比較好

今天會示範如何使用Angular Cli來建立一個元件並且發佈到npm

<!--more-->

# 產生Component

1. 使用cli產生一個專案

```shell
ng new my-component
```

2. 建立一個獨立的`module`叫做first

```shell
ng g m first
```

>給別人的元件一定是獨立的module，除非你要將`AppModule`當做給人import的module，但一般我們不會這樣做，而是弄一個相對有意義的名稱

3. 產生一個`component`叫做red

```shell
ng g c first/red
```

4. 這個功能我們就先讓component的字顏色變成紅色

```html
<p style="color:red">
  <ng-content></ng-content>
</p>
```

5. 接著我們先用一般的方式來看看有沒有如我們預期

```html
<app-red>this is my first component</app-red>
```

# 封裝

這邊我們使用`ng-packagr`的套件來協助我們達到這件事情

> npm i ng-packagr --save-dev

接著我們要產生兩個檔案，首先是`ng-package.json`，要注意一定要有`whitelistedNonPeerDependencies`將一些angular已經包含的東西都過濾掉不打包

```
{
  "$schema": "./node_modules/ng-packagr/ng-package.schema.json",
  "lib": {
    "entryFile": "public_api.ts"
  },
  "whitelistedNonPeerDependencies": [
    "."
  ]
}
```

再來要產生`public_api.ts`，這個檔案會描述所有要對外開放的module

```typescript
export * from './src/app/first/first.module';
```

這些前置作業都做完後，我們就可以來進行編譯，那這邊可以偷懶一點在package.json加上指令

```
{
    "scripts":{
        "start": "ng serve",
        "packagr": "ng-packagr -p ng-package.json"
    }
}
```

執行完`npm run packagr`的指令後，就可以看到增加了一個`dist`的目錄

# 測試

上面的動作都執行完以後，我們可以先進行個簡單的測試，用另一個專案將剛剛打包的東西引入

首先要先產生一個可以被npm認識的tgz檔案，所以要先切換到`dist`的目錄，並且執行指令`npm pack`，就可以看到`dist`目錄中會多出`my-component-0.0.0.tgz`的檔案，這就是要放到另一個專案中被install的檔案

> 0.0.0是根據package.json的version而來

建立了一個新的專案`ng new test-component`然後把剛剛產生的檔案放到根目錄，並且執行`npm install my-component-0.0.0.tgz`，就可以開始使用

首先在`AppModule`加上我們所建立的Module

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FirstModule } from 'my-component';
import { AppComponent } from './app.component';


@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FirstModule //我們所建立的module
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

再來就是在`app.component.html`中加上`app-red`

```html
<app-red>hello component</app-red>
```

執行起來就可以看到畫面上出現`hello component`並且呈現紅色，那這個功能就是沒問題

> 當然也可以把這個步驟放到發佈之後，直接從npm上面下載下來

# 發佈

這個地方就很簡單啦，只要在`dist`的目錄下執行`npm publish`就可以發布到npm中囉！

只要跟平常我們安裝新的套件方式一樣，執行`npm install my-component`就可以

> 記得先要有npm的帳號，並且在本機登入

# 參考

[Building an Angular 4 Component Library with the Angular CLI and ng-packagr](https://medium.com/@nikolasleblanc/building-an-angular-4-component-library-with-the-angular-cli-and-ng-packagr-53b2ade0701e)

另一個套件的資料也可以參考

[How to build and publish an Angular module](https://medium.com/@cyrilletuzi/how-to-build-and-publish-an-angular-module-7ad19c0b4464)

# 補充

2018/06/20 在線上讀書會中分享，kevin也提到`angular cli 6`的版本已經包含這個功能

```shell
ng g library jimmy-demo
```

執行後，可以看到整個目錄中多了一個`projects`的目錄，可以看到是一個完整的專案，編譯的話就要特別指定

```shell
ng build jimmy-demo
```

一樣可以在`dist`中看到一個目錄`jimmy-demo`，只要切到這個目錄底下就可以做`npm publish`