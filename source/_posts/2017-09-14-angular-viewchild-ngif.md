---
title: '[Angular] ViewChild遇上ngIf'
date: 2017-09-14 15:20:01
categories:
- Front-end
- Angular
tags:
- Angular
---

當我們有一個child component想要用ViewChild的方法在parent component取得物件，但如果在上面加上ngIf的時候，取得物件的這個動作就會發現根本取不到!!!

<!--more-->

在app.component中，使用todo.component，並且加上ngIf作為控制是否顯示

```html
<button (click)="setFlag(!flag)">toggle flag</button>

<app-todo *ngIf="flag">
</app-todo>
```

在component中，使用`ViewChild`的方法並且在`setFlag`中印出物件，就可以發現在這個時候是取不到todo這個物件的，就算是用angularJS常用的setTimeout也是一樣

這個問題應該就是變數還沒反應到html，所以導致根本還沒拿到這個物件

```typescript
export class AppComponent {
  @ViewChild(TodoComponent) todo: TodoComponent;

  setFlag(flag) {
    this.flag = flag;
    console.log('a', this.todo);
    setTimeout(function() {
      console.log('b', this.todo);
    }, 10);
  }
}
```

# 解決方案

但這樣根本不能拿到物件要怎麼往後處理，那這邊有兩個作法

1. 在child component使用事件作為觸發，當物件被ngIf啟動後，就可以拋出事件

   ```typescript
   export class TodoComponent implements AfterViewInit {
     @Output() onInit = new EventEmitter();

     ngAfterViewInit() {
       this.onInit.emit(this);
     }
   }
   ```

   在parent component收到事件後再去操作ViewChild的物件，就可以正常使用

   ```html
   <button (click)="setFlag(!flag)">toggle flag</button>

   <app-todo (onInit)="trigerTodo($event)" *ngIf="flag">
   </app-todo>
   ```

   ```typescript
   export class AppComponent {
     @ViewChild(TodoComponent) todo: TodoComponent;

     setFlag(flag) {
       this.flag = flag;
     }

     trigerTodo(event) {
       console.log('component', event);
     }
   }
   ```

2. ViewChild使用setter，當物件被建立時，就可以在裡面進行操作

   ```typescript
   export class AppComponent {
     @ViewChild(TodoComponent) set todo(content: ViewContainerRef) {
       console.log('setter', content);
     }

     setFlag(flag) {
       this.flag = flag;
     }
   }
   ```

# 結論

其實在使用ViewChild的時候，使用ngIf其實是不好的應用，但如果真的要用就必須要繞路一下，也要去了解一下各種使用方法，上面的兩個方法，我個人比較喜歡setter的作法，簡單乾淨!

如果有其他作法或想法歡迎來討論喔!!