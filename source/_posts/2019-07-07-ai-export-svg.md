---
title: AI匯出svg小雷
date: 2019-07-07 11:28:17
categories:
- Debug
tags:
- Debug
- SVG
- AI
---

AI全名是Adobe Illustrator，由Adobe出的一個繪圖工具，今天為什麼會講到這個工具呢？
原因是最近公司要做re-brand的動作，designer出的icon都是用svg做匯出，但卻發生了顏色跑掉的問題，原本是**藍色**變成**黑色**

<!--more-->

> 版本介紹
>
> * Adobe Illustrator 2017 cc中文

# 簡單介紹SVG

SVG全名是Scalable Vector Graphics，透過xml的格式撰寫，可縮放的向量圖。因為是直接把顏色當成是attribute的方式做宣告因此同一個svg要呈現不同顏色時不用再重出一張圖，只要透過css就可以做到染色的行為。

# 問題檔案內容

剛剛有說到顏色可以透過attribute的方式宣告，並且可以透過css的方式做染色的行為，今天這個狀況就是發生在css的部份

```xml
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
  <defs>
    <style>.cls-1{fill:#231714;}</style>
	</defs>
	<title>ic_audio_pause_240</title>
  <path class="cls-1" d="M12,1A11,11,0,1,0,23,12,11,11,0,0,0,12,1Zm0,20a9,9,0,1,1,9-9A9,9,0,0,1,12,21Z"/>
  <rect class="cls-1" x="9" y="8" width="2" height="8" rx="0.5"/>
  <rect class="cls-1" x="13" y="8" width="2" height="8" rx="0.5"/>
</svg>
```

因為是透過AI產生的圖檔，class的名稱對軟體來說不重要，只要有產生預期的效果就好，因此所有的icon都是從`.cls-1`開始做命名，然後輸出的時候又是用style的方式做顏色的宣告。

## 問題解析

可能有人發現問題點了，**<style></style>**是一種全域的宣告，也就是說今天你的html中只要有任何一個class叫做`.cls-1`的全部都會符合這個selector被染成紅色

再加上css的特性之一，相同的宣告後面的會蓋掉前面的，也就是常在說的覆蓋，因此只要`.cls-1`這個class的顏色有不一樣，那就會變成看誰是最後被載入的，全部都會跟著變成他的顏色

請看[完整範例](https://codepen.io/jiaming0708/pen/dBZZmm)

# AI怎麼設定

這個問題當然只要把svg修掉就解決，問題是每次designer出圖的時候都要在那邊檢查修改嗎？難道AI沒有辦法從輸出的時候就處理掉這個問題嗎？

首先來看一下AI輸出的畫面，點選工具列 **檔案** > **轉存** > **轉存為螢幕適用**

就可以看到下方這個畫面，右側選擇格式的右上方有一個小小齒輪（紅色圈）

![export](export.png)

點進去以後可以看到下面這個畫面，接著選到**SVG**看一下裡面有哪些設定

![export setting](export_setting.png)

這次的目標是要把樣式改對，因此點選樣式設定，可以看到有三個選項

* 內部CSS <== 預設
* 內嵌樣式
* 簡報屬性 <== 請選擇

> 是說有沒有人覺得**簡報屬性**的翻譯很難懂阿，我看到的當下完全不知道是要做什麼用...lol

將樣式改為簡報屬性，再次輸出測試，可以看到顏色就變成使用attribute做宣告，這樣就可以打完收工

```xml
<svg id="Layer_1" data-name="Layer 1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
  <title>ic_audio_pause_240</title>
  <path class="cls-1" fill="#231714" d="M12,1A11,11,0,1,0,23,12,11,11,0,0,0,12,1Zm0,20a9,9,0,1,1,9-9A9,9,0,0,1,12,21Z"/>
  <rect class="cls-1" x="9" y="8" width="2" height="8" rx="0.5"/>
  <rect class="cls-1" x="13" y="8" width="2" height="8" rx="0.5"/>
</svg>
```

# 結論

這個問題真的很難遇到(?)，要不是公司要做re-brand，平常designer畫UI出mockup時用**sketch**這套軟體產生的檔案也都沒問題，但本著從根本解決問題，遇到的當下就拉著designer一起坐下來看有什麼地方是可以調整

> 另外補充一個小雷，很多人用svg來產生圖檔之類，有的圖可能包含著漸層，如果說把圖sketch再輸出的話，漸層就會有部分被吃掉，如果是直接輸出的話就沒有這個問題喔