---
title: camera照片大小設定
date: 2016-11-29 17:07:11
categories:
- APP
- Cordova
tags:
- Cordova
- ionic
---

最近在用ionic1.x版的開發APP，其中需要一個拍照的功能，使用套件[`cordova-plugin-camera`](https://github.com/apache/cordova-plugin-camera)
那照片會在最後一起上傳到server，但是現在的手機像素其實很高
導致一兩張照片而已就已經爆掉無法傳送（當然也可能是我不懂得怎麼寫上傳照片啦，我是轉成base64後上傳）

一開始沒仔細看官方文件（這很重要記得要好好看）
就去搜尋了image resize的作法，得到使用canvas轉換，這個作法另外開一篇寫好了
這個作法很可怕，圖片縮小後光線強的地方直線都變成鋸齒狀

就回套件看看有沒有什麼作法

| column | type | description |
|:-------|:----:|:------------|
| targetWidth | number | Width in pixels to scale image. Must be used with targetHeight. Aspect ratio remains constant. |
| targetHeight | number | Height in pixels to scale image. Must be used with targetWidth. Aspect ratio remains constant. |

這邊有說到可以等比例，好像是我要的，但是怎麼設定才會等比例阿...
看不到太懂說明，只好打開java檔來看看

``` java
if (targetHeight > 0 && targetWidth > 0 && targetWidth == targetHeight) {
    intent.putExtra("aspectX", 1);
    intent.putExtra("aspectY", 1);
}
```

原來只要給一樣的值，就會等比例的縮放，下面就是我的範例啦

``` js
navigator.camera.getPicture(onSuccess, onFail, {
  quality: 100,
  destinationType: Camera.DestinationType.DATA_URL, //FILE_URI、DATA_URL
  targetWidth: 500,
  targetHeight: 500
});
```