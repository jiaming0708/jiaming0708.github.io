---
title: Vue初體驗
date: 2018-05-21 22:04:31
categories:
- Frontend
- Vue
tags:
- Vue
---

最近朋友要做一個公司的官網，身為前端人（自己說XD），怎麼可以用wix/wordpress那些啦一啦而已呢
當然要玩一下沒玩過的東西，一直聽說Vue不錯用，因此就來玩玩看囉

<!--more-->

## 建立專案

起手式，當然要先來個[vue-cli](https://github.com/vuejs/vue-cli)

```powershell
npm install -g @vue/cli
```

接著就是要建立專案

```powershell
vue create my-project
```

這邊提供兩種方式建立專案，個人建議用manually，原因等一下可以看到

![create](create.png)

### defaul

首先來看一下default，就是一個很簡單的內容，沒有route沒有scss，但我們通常會希望有route、scss這些基本的東西吧，因此還是要用進階的設定

![default](default.png)

### manually

這時候我們可以選擇一些想要使用在這個專案中的功能，像是`TypeScript`、`Router`、`SCSS`，這些是我基本會用到的，其他就是要看專案的性質囉

![manually1](manually1.png)

接著會根據你的選擇，會有不同的設定，像是這樣

![manually2](manually2.png)

產生出來的專案就會是這樣，是不是比較符合我們要的結果了，可以開工囉！

![manually3](manually3.png)

## 開始享受

`Vue`的整個生命週期是從`main.ts`開始，從這邊可以看到透過`render`的方式將`App`這個內容產生出來，然後替換#app`的dom

```javascript
import Vue from 'vue';
import App from './App.vue';
import router from './router';

new Vue({
  router,
  render: (h) => h(App),
}).$mount('#app');
```

順著內容看下去，接著來看看`App.vue`是在做什麼的，`.vue`這個副檔名就是component的宣告內容，其中分為三塊

### template

* 一個component必須要有一個element作為root，可以參考下面的範例

```html
<!--correct-->
<template>
	<div></div>
</template>
<!--error-->
<template>
    <div></div>
    <div></div>
</template>
```

* 透過大括號的方式`{{title}}`把model的資料放進畫面中
* 使用`v-*`開頭對element做變化的語法，例如`v-for`就是重複element的指令；其他指令就去官網查詢囉

### script

 一個component不一定要有這段，但如果需要有model或是引用其他component就必須要加上`script`

* 要被引用的component必須要被放在`components`中，而且是用物件的方式來撰寫
* 在這個component自己宣告的model必須要用`data`宣告，而且必須是一個function

```javascript
import Vue from 'vue';
import Header from '@/components/Header.vue';
import Footer from '@/components/Footer.vue';

export default Vue.extend({
  name: 'app',
  components: {
    Header,
    Footer,
  },
  data: () => {
      return {
          title: 'Hello'
      };
  },
});
```

### style

有兩個方法可以來定義css的範圍

* 向下繼承，像是`App`這個component的定義就會是這種模式，但其他地方不建議使用，很容易污染（這邊我不太喜歡比較喜歡把這種全域的另外拉出去一個檔案，然後直接被index.html引用，但要自己去寫webpack處理....懶）
* component限定，編譯的時候會加上一個id之類的在所有的css，來確保是只有這個component使用到

```html
<style scoped lang="scss">
</style>
<style lang="scss">
</style>
```

> 在這邊我喜歡把scss另外開檔案寫，比較乾淨一點

### router

看完了`.vue`的檔案結構以後，已經有些了解，接著就是來看看SPA一定會遇到的`route`是怎麼定義和使用

首先必須要先new一個Router的物件，然後再裡面定義`path`、`name`、`component`
還記得`App.vue`中有import一個`router`嗎？其實就是引入這個地方的設定

```javascript
import Vue from 'vue';
import Router from 'vue-router';
import Home from './views/Home.vue';
import About from './views/About.vue';

Vue.use(Router);

export default new Router({
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home,
    },
    {
      path: '/about',
      name: 'about',
      component: About,
    },
  ],
});
```

接著在template使用`router-link`就可以

* `to`也就是url，前面檔案所定義的`path`
* `active-class`當目前url跟目前route一樣時，就會套上這個class，預設class是`router-link-active`
* `exact`是為了讓`active-class`運作更正確一點，當目前到`/about`的路徑，則`/`和`/about`都會被套上class，但我們預期應該是只有一個被套用，因此要將這個屬性設定上去（簡單說就是完全符合path）

```html
<router-link to="/about" active-class="selected" exact>
</router-link>
```

## 結論

用一個簡單的網站練習一下vue，其實內容整體來說不差，真的不能否認小專案用vue上手還蠻快速的，如果真的要到專案/產品中使用，還有很多東西需要摸索

因此對我來說`Angular`更適合一點，兩個的寫法上差異真的沒有很大，但是cli的方便性或是檔案的分離，都讓我更加的喜歡（拜託不要來戰我～～）

> 另外有幾個地方想請教一下大家為什麼要這樣用
>
> * 使用export default而不是給一個實際名稱
> * 把template、script、style都寫在一起，這樣如果這個功能稍微複雜一點，不就會很「漏漏等」，還是說這應該算是component的設計失敗?