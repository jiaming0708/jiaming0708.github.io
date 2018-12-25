---
title: crop系列2-canvas drawimage應用
date: 2018-12-22 10:32:17
categories:
- Front-end
- canvas
tags:
- canvas
- crop
---

這是接續前一篇{% post_link css-scale-position %}的系列之二，前面有提到說要做圖片剪裁，經過了drag和scale後，只要留下一個區塊是我們要的，這時候canvas就出現啦

<!-- more -->

首先我們會把圖片先放在`img`的element中，然後再讓使用者drag和scale後，只裁切圓形所顯示的部份，但我們不會真的裁切成圓形，而是方形作為儲存

# 不是你以為的樣子

首先我們有一個img的elment，實際大小是`300*227`，指定寬度為200，下面是實際呈現出來的樣式

```html
<img src="https://mdn.mozillademos.org/files/5397/rhino.jpg" width="200" id="image" />
```

{% img https://mdn.mozillademos.org/files/5397/rhino.jpg 200 河馬 %}

接著我們要開始使用canvas，把圖片放進去，直接先指定畫布大小是`200*200`

```javascript
const img = document.querySelector('#image');
const canvas = document.createElement('canvas');
canvas.width = 200;
canvas.height = 200;
const ctx = canvas.getContext('2d');
ctx.drawImage(img, 0, 0);
```

你可以發現到一件事情，圖被切割了！！畫出來的大小竟然不是跟image的元件所產生的是一樣大，而是根據原始檔案

<iframe style="border:none" width="100%" height="450px" src="https://stackblitz.com/edit/canvas-version1?embed=1&file=src/app/app.component.html&hideExplorer=1&view=editor"></iframe>

# 指定區域截取

接下來，就是要開始做截取圖片，在`drawImage`有提供override可以使用，要使用的是其中最多參數的那個，那要先知道每個參數的意義，可以參考一下[這篇](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage)的說明

放一個`50*50`的紅框，在畫面上，作為識別我們預期要截取的地方

```css
.block{
  width: 50px;
  height: 50px;
  border: 1px solid red;
  position: absolute;
  top: 10px;
  left: 100px;
}
```

在draw的地方改一下參數

```javascript
ctx.drawImage(img, 10, 100, 50, 50, 0, 0, 50, 50);
```

<iframe style="border:none" width="100%" height="450px" src="https://stackblitz.com/edit/canvas-version2?embed=1&file=src/app/app.component.ts&hideExplorer=1&view=editor"></iframe>

完蛋了！取到的地方跟我們預期完全不一樣，其實這跟上面所提到的canvas是用實際大小有關係，所以這時候就要來做一點運算，讓位置是符合我們的預期

原圖是`300*227`，實際是`200*151`，因此位置要跟著等比例放大，因此先將比例算出來，再拿來計算

```javascript
const {
      offsetWidth,
      naturalWidth,
} = img;
const ratio = naturalWidth / offsetWidth;
const {
    offsetTop,
    offsetLeft,
} = document.querySelector('.block');
const realTop = offsetTop * ratio;
const realLeft = oofsetLeft * ratio;
const realWidth = blockWidth * ratio;
const realHeight = blockHeight * ratio;
ctx.drawImage(img, realLeft, realTop, realWidth, realHeight, 0, 0, 50, 50);
```

<iframe style="border:none" width="100%" height="450px" src="https://stackblitz.com/edit/canvas-version2?embed=1&file=src/app/app.component.ts&hideExplorer=1&view=editor"></iframe>

耶，終於成功啦！拿到我們要的區域，那我們就可以匯出成圖片囉

# 匯出圖片

我們要把截取出來的區域變成blob，最簡單的方法可以使用`toBlob`，但是要注意在`safari`是不支援的，那這邊有兩個方法，一個是用polyfill，那另一個就是用`toDataURL`的作法

這邊介紹的我用的方法`toDataURL`轉成blob以後丟給api給後端處理

```javascript
// 先輸出成為base64的字串
const image = canvas.toDataURL();
// 將base64的資料還原成array
const byteString = atob(image.split(',')[1]);
// 轉換成byte
const ab = new ArrayBuffer(byteString.length);
const ia = new Uint8Array(ab);
for (let i = 0; i < byteString.length; i += 1) {
  ia[i] = byteString.charCodeAt(i);
}
const newBlob = new Blob([ib], {
  type: 'image/jpeg',
});
```

# 結論

一開始弄的時候被size問題困擾好久，因為還加上了scale的功能，然後怎麼拿到的圖片都不對，後來才發現原來又要拿數學來做計算，然後後來又被一個`toBlob`搞到，所以才又生出一段轉換的動作

# 參考

* [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Using_images)

* [base64 to blob](https://stackoverflow.com/questions/4998908/convert-data-uri-to-file-then-append-to-formdata)