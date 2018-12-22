---
title: scale和position的糾葛-crop(1)
date: 2018-12-20 22:27:56
categories:
- Front-end
- CSS
tags:
- CSS
---

最近剛做完一個功能是圖片上傳剪裁(要支援drag和scale)，晚點會把全部整理起來，先來分享一個小東西
當我們有一個block要做scale，那這時候的position會是？

<!--more-->

# 範例

> 使用angular作為樣本，只是因為懶得寫insert值到element中XD

先來看看一開始還沒做scale前，我們的外框距離top是50px並且水平置中，裡面的話是距離外框top是31px，left是21px

<iframe style="border:none" width="100%" height="450px" src="https://stackblitz.com/edit/scale-position?embed=1&file=src/app/app.component.ts&view=preview"></iframe>

根據這樣的條件，我們可以先來想像一下，當內框用`transform`放大的時候，我們要拿到的offset、client各會是多少？

> 可以再editor中改`app.component.scss`的`.scale`，已經有先放了一個`transform: scale(1)`在裡面

# 答案

其實答案不難，首先要知道`[transform](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)`是屬於效果，而並不是實際element的width、height放大，所以**offset是不會改變**！

但是client呢？就可以透過`getBoundingClientRect`這個方法，拿到我們所預期的值！

# 進階

但你也許會跟我一樣疑惑，client跟offset是完全不一樣的資料，難道我沒有辦法拿到scale後的offset是多少嗎？
當然也還是有辦法，這時候我們要透過公式去計算他（一定要寫一點程式，不然有點難過XD）

1. 拿到原始的width、height，並且乘上scale值

   ```javascript
   const widtd = 100;
   const height = 100;
   const scale = 1.5;
   const realWidth = width * scale; // 150
   const realHeight = height * scale; // 150
   ```

2. 因為scale是以中心等比縮放，所以這邊要將計算後和計算前的數值相減除以2

   ```javascript
   const diffX = (realWidth - width) / 2; // 25
   const diffY = (realHeight - height) / 2; // 25
   ```

3. 這個數值就會是top和left的偏移量，只要將原始的座標減掉偏移量即可

   ```javascript
   // before
   let position = {
       left: 50,
       top: 10,
   };
   // after
   position = {
       left: position.left - diffX,
       top: psoition.top - diffY,
   };
   ```

那來總結一下這個算式，`(scale - 1) / 2 * value`這就是取得偏移量的公式！

> 2018/12/22調整title和第一段描述