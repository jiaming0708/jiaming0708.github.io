---
title: '[Angular] Directive介紹'
date: 2017-08-16 10:32:42
categories:
- Frontend
- Angular
tags:
- Angular
---

為什麼要用Directive，可以幫助我們做到什麼事情?

<!--more-->

directive是一個可以擴充element的行為或是呈現，最常使用的component則是繼承了directive加上template的部份，

這時候我們可以使用directive來幫助我們，把重複的東西抽離出來，並且能夠讓element變得精簡

directive有三種類型

- Component
- Structural Directive
- Attribute Directive
## Structural Directive

對HTML的結構進行操作，像是隱藏、顯示或是迴圈等的動作。
常用的功能有`ngIf`、`ngFor`、`ngSwitch`。

`*ngIf`代表著是否要顯示dom，這邊隱藏的話是不會去真正的產生element

```
    <div *ngIf="true">this is show</div>
    
    <div *ngIf="false">this is hide</div>
    <!--template bindings={
      "ng-reflect-ng-if": null
    }-->
```

`*ngFor`迴圈顯示，被包在裡面的html可以自行定義

> 這邊要注意，當變數是null則會死掉，這時候外面可以包一個*ngIf

```
    <div *ngFor="let todo of todoList">
      {{todo.name}}
    </div>
```

`ngSwith`樣板有多種需要動態決定顯示哪個時可以使用
```
    <div [ngSwitch]="hero?.emotion">
      <happy-hero    *ngSwitchCase="'happy'"    [hero]="hero"></happy-hero>
      <sad-hero      *ngSwitchCase="'sad'"      [hero]="hero"></sad-hero>
      <confused-hero *ngSwitchCase="'confused'" [hero]="hero"></confused-hero>
      <unknown-hero  *ngSwitchDefault           [hero]="hero"></unknown-hero>
    </div>
```

使用*做為前置詞，用來跟其他功能做區隔
當然也可以用其他的寫法
```
    <div template="ngIf hero">{{hero.name}}</div>
    
    <ng-template [ngIf]="hero">
      <div>{{hero.name}}</div>
    </ng-template>
```

## Attribute Directive

可以改變element的行為或是樣式

`ngClass`改變element的class，使用陣列的方式或用物件的方式來決定要哪些class要被引用
```
    <div [ngClass]="'selected'">
    </div>
    <div [ngClass]="['selected']">
    </div>
    <div [ngClass]={'selected':true}>
    </div>
```
`ngStyle`改變element的style內容，與ngClass的用法相同


## 動手做

 現在可以來試試看寫個attribute directive
```
    import { Directive, ElementRef, Input } from '@angular/core';
    
    @Directive({ selector: '[myTodo]' })
    export class TodoDirective {
        constructor(el: ElementRef) {
           el.nativeElement.classList.add('completed');
        }
    }
```
**selector用法**

- `element-name`
- `.class`
- `[attribute]`
- `[attribute=value]`
- `:not(sub_selector)`
- `selector1, selector2`
## @Input/@Ouput

component之間或是和directive要互相傳遞資料的時候，我們能夠透過input和output的方式來做傳遞。
接著來看看如何使用
```
    @Directive({select: '[myTodo]' })
    export class TodoDirective{
      //這是相同的意思
      @Input() todo:string;
      @Input('status') _status:boolean;
      //Output只能拋出event喔!
      @Output() onEventChange = new EventEmitter();
    }
```
像上面的例子，如果希望把傳進來的status來改變style狀態，可以這樣做
```
    @Directive({select: '[myTodo]' })
    export class TodoDirective{
      @Input('status') _status:boolean;
      
      constructor(el: ElementRef){
        if(this._status){
          el.nativeElement.classList.add('complete');
        } else {
          el.nativeElement.classList.remove('complete');
        }
      }
    }
```
但其實這樣很麻煩，而且只有第一次產生物件時會執行，可以改成用HostBinding的方法來做
```
    @Directive({select: '[myTodo]' })
    export class TodoDirective{
      @Input('status') @HostBinding('class.completed') _status:boolean;
      
      constructor(el: ElementRef){
      }
    }
```
@HostBinding是用來變更directive所在的那個element的一些屬性，例如：style、class等，那你也許會問可以變更屬性，那能不能監聽element的事件呢？答案是可以的，我們可以來用用@HostListener

```
    @Directive({select: '[myTodo]' })
    export class TodoDirective{
      @Input('status') @HostBinding('class.completed') _status:boolean;
      //Output只能拋出event喔!
      @Output() onEventChange = new EventEmitter();
      @HostListener('click') onHostClick(){
        this.el.nativeElement.classList.toggle('complete');
        this.onEventChange.emit();
      };
      
      constructor(private el: ElementRef){
      }
    }
```

## 參考資料

[官網](https://angular.io/guide/attribute-directives)
[Jeff筆記](https://jeffwu85182.github.io/2017/03/25/angular-directive-reaserch/)

