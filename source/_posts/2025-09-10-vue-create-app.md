---
title: Vue 的 CreateApp 寫法和比較
date: 2025-09-10 09:34:29
updated: 2025-09-10 09:34:29
categories:
- Frontend
- Vue
tags:
- Frontend
- Vue
thumbnail: 
---

最近在公司比較舊的架構 MVC + Vue，發現可以不要有 RootComponent 的宣告，並且還會根據 html 上有沒有那個 component 決定是不是要 render，這樣的做法在 React/Agnular 的世界中都沒看過，因此很好奇的去研究了一番。

<!-- more -->

## 寫法
Vue 的 CreateApp 寫法官方有提供幾種

### 使用 DOM

使用 `#app` 的 element 作為入口，直接針對裡面的內容作解析

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>

<div id="app">{{ message }}</div>

<script>
  const { createApp, ref } = Vue

  createApp({
    setup() {
      const message = ref('Hello vue!')
      return {
        message
      }
    }
  }).mount('#app')
</script>
```

### RootComponent

宣告一個 RootComponent 置換 `#app` 裡面的內容，以下面的內容來說看不到 `test` 的字樣在畫面上，而只會看到 `Count`

```html
<div id="app">test</div>
<script>
  const { createApp, ref } = Vue

  const MyComponent = {
    setup() {
      const count = ref(0)
      return { count }
    },
    template: `<div>Count is: {{ count }}</div>`
  }
  createApp(MyComponent).mount('#app')
</script>
```

### Multiple Component

宣告多個 Component 必須在 html 先寫好才會 render，以範例來說會看到 `test` 以及 `Count` 的字樣於畫面上

```html
<div id="app">
    test
    <My-Component></My-Component>
</div>
<script>
    const { createApp, ref } = Vue
    const MyComponent = {
      setup() {
        const count = ref(0)
        return { count }
      },
      template: `<div>Count is: {{ count }}</div>`
    }
    const app = createApp({
      components: {
        MyComponent,
      },
      setup() {
      }
    })
    app.mount('#app')
</script>

```

## 寫法比較

於是我就很好奇這三種寫法有沒有比較適合的情境，畢竟我沒有寫過 Vue 所以請 ChatGPT 來協助我

| 寫法                       | 範例                                                                                                                    | 優點                                                        | 缺點                                   | 適用情境             |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------ | ---------------- |
| **單純掛載 DOM**          | app.mount('#app')                                                                                         | - 最簡單<br>- 快速啟動 Vue                                       | - Root component 不明確<br>- 管理性差，不適合擴展 | 快速測試、小範例         |
| **直接指定單一元件為 root**    | `import MyComponent from './my-component.js'`<br>`createApp(MyComponent).mount('#app')`                       | - 精簡、乾淨<br>- 不需要額外定義 root                                 | - 不易擴展<br>- 缺少統一入口結構                 | 小型 widget 或單頁小工具 |
| **自訂 root component** | `const app = createApp({  components: { MyComponent },  setup() { ... }})`<br>`app.mount('#app')` | - 彈性高<br>- 容易擴展，適合加 router/store<br>- 結構清晰，可轉換成 `App.vue` | - 稍微多些樣板<br>- 小專案可能顯得冗餘              | 中大型專案，需集中管理      |


## 目前公司的作法

回到一開始為什麼我會對這樣的寫法充滿著不解，一開始查的時候頁面沒有打出某支 API，才發現原來是條件不滿足所以沒有 render 那個元件在 html 上而導致。
概念大概像是這樣

```html
<div id="app">
    <Component1></Component1>
    <!-- <Component2></Component2> -->
    <Component3></Component3>
    <Component4></Component4>
    <Component5></Component5>
    <Component6></Component6>
</div>
```

```js
    const app = createApp({
      components: {
        Component1,
        Component2,
        Component3,
        Component4,
        Component5,
        Component6,
      },
      setup() {
      }
    })
```

從 MVC render 出來的 html 沒有 `Component2`，那隻沒有被打出來的 API 就是包在這個元件中。

嘗試著去思考前人這樣寫會有什麼樣的好處，後來想到一個可能性，因為有多個頁面有些元件是共用的，用了這個寫法，就可以不用宣告一堆 `RootComponent` 給不同頁面，只要根據不同頁面需要的 component 套進去就好，有點像是積木一樣的方式去組合，增加了一些彈性以及維護性。