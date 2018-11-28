---
title: Angular HostBinding&HostListener
date: 2017-03-27 17:33:00
categories:
- Front-end
- Angular
tags:
- Angular
---
## 介紹
今天要來講講HostBinding和HostListener，原本官網還能找到文章，但最近才發現他已經被殺了...
但還是要來說一下，誰叫我自己挖洞給自己跳了o(〒﹏〒)o

<!--more-->

## 目標
來做一個highlight的功能，顏色可以自定，當游標進入時變色，離開則回復
如果是寫jquery，你會怎麼做?  使用on監聽事件

``` js
let color = 'red';
$(document).on('mouseenter', '.class', function(){
    $(this).style('background-color', color);
});
$(document).on('mouseleave', '.class', function(){
    $(this).style('background-color', '');
});
```

但我們現在是寫angular，當然要想辦法變成directive
並且能做到像jquery的事情一樣

## 建立directive
首先，先建立一個directive，並且傳入顏色
``` typescript
import { Directive, ElementRef, Input, Component } from '@angular/core';
@Directive({
  selector: '[highlight]'
})
export class HighlightDirective {
  private _defaultColor: string = 'red';

  constructor(private el:ElementRef) {
  }

  @Input('color') color: string;
}

@Component({
  selector: 'app-root',
  template: `
    <div class="highlight">
        Highlight Me!!
    </div>
  `
})

export class AppComponent {
}
```

## HostListener
接著開始監聽事件，但是要怎麼做到，從一個element的屬性去監聽他的行為
這時候我們需要用到HostListener，從字面其實可以猜的出來，他就是用來監聽來自element(host)

``` typescript
import { Directive, ElementRef, Input, HostListener } from '@angular/core';

@Directive({
  selector: '.highlight'
})
export class HighlightDirective {
  private _defaultColor: string = 'red';


  constructor(private el:ElementRef) {
  }

  @Input('color') color: string;

  @HostListener('mouseenter') onEnter() {
    this.el.nativeElement.style.backgroundColor = this.color || this._defaultColor;
  }

  @HostListener('mouseleave') onLeave() {
    this.el.nativeElement.style.backgroundColor = null;
  }
}
```

> 補充說明:
beta版本時(遙遠的古代...)，是在directive那邊的描述中加上一個host的來使用
```typescript
@Directive({
 selector: '[highlight]',
 host: {
   '(mouseenter)': 'onMouseEnter()',
 }
})
```

## HostBinding
可是上面那樣寫還是不夠漂亮，還要在事件內去控制，為什麼不能用model去控制呢
因此angular有提供HostBinding的功能，讓你可以直接用model來影響element屬性

```typescript
import { Directive, ElementRef, Input, HostListener, HostBinding } from '@angular/core';

@Directive({
  selector: '.highlight'
})
export class HighlightDirective {
  private _defaultColor: string = 'red';

  constructor(private el:ElementRef) {
  }

  @Input('color') color: string;
  
  @HostBinding('style.backgroundColor') bgColor: string;
  @HostListener('mouseenter') onEnter() {
    this.bgColor = this.color || this._defaultColor;
  }

  @HostListener('mouseleave') onLeave() {
    this.bgColor = null;
  }
}
```
哇屋～這樣的code看起來是不是更為清楚，而且好操作

這個範例是變更style的屬性，那如果要變更class的話，可以用下面的方法
使用boolean來控制class是否被引用
``` typescript
@HostBinding('class.draging') isDraging: boolean = false;
```

來點進階的內容，input和hostBinding合在一起有沒有搞頭
答案是，絕對有搞頭
``` typescript
@Input('color')
@HostBinding('style.backgroundColor')
bgColor: string;
```

外面的element用color這個屬性來傳入顏色，放入到bgColor這個變數，然後來影響backgroundColor
這樣可以讓變數減少，並且整合在一起，閱讀起來也很清楚

## 加上Output
上面的應用其實非常的簡單，但通常不會有這麼簡單的應用，那接著我們來加點料吧
在進入時，我們需要拋出event，讓component呈現計數滑動次數吧！

``` typescript
import { Directive, ElementRef, Input, HostListener, HostBinding, Output, EventEmitter } from '@angular/core';

@Directive({
  selector: '.highlight'
})
export class HighlightDirective {
  private _defaultColor: string = 'red';

  constructor(private el:ElementRef) {
  }

  @Input('color') color: string;
  @Output() onColorChange = new EventEmitter;
  
  @HostBinding('style.backgroundColor') bgColor: string;
  @HostListener('mouseenter') onEnter() {
    this.bgColor = this.color || this._defaultColor;
    this.onColorChange.emit();
  }

  @HostListener('mouseleave') onLeave() {
    this.bgColor = null;
  }
}

@Component({
  selector: 'app-root',
  template: `
    <div class="highlight" (onColorChange)="onColorChange()">
        Highlight Me!!
    </div>
    <div>
        {{enteryCount}}
    </div>
  `
})

export class AppComponent {
    enteryCount = 0;
    onColorChange(){
        this.enteryCount++;
    }
}
```

## Reference
https://ng2.codecraft.tv/custom-directives/hostlistener-and-hostbinding/
https://angular.io/docs/ts/latest/guide/cheatsheet.html
https://angular.io/docs/ts/latest/guide/attribute-directives.html