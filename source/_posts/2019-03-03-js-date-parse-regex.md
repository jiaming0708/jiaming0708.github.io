---
title: '[JS] date自定義格式'
date: 2019-03-03 10:35:15
updated: 2019-03-03 10:35:15
categories:
- Frontend
- JavaScript
tags:
- JavaScript
- regex
---

前幾天收到QA開的一個bug，是關於日期時間顯示的錯誤，因為開發習慣都是以chrome為主，所以沒特別發現問題，結果QA是在firefox使用而發現的bug，經過追查以後才發現是字串轉成日期格式有問題

<!-- more -->

# 日期格式

從後端來的資料格式是`2019-03-03 10:35:15 +0800`，如果用chrome可以正確的解析出來，但如果是用firefox、safari則會出現`NaN`代表著轉換失敗，遇到這種狀況就是先去查一下文件，確認標準格式是什麼，並且測試看看怎樣的轉換是可以work的

根據[MDN文件](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/parse#Date_Time_String_Format)表示，支援三種ISO格式

> 這邊不使用GTM，是因為資料格式差異比較大，轉換上更麻煩

* 只有日期`2011-10-10`
* 日期加時間`2011-10-10T14:48:00`
* 毫秒加時區`2011-10-10T14:48:00.000+09:00`

從這邊可以知道後端回來的格式跟標準還是有所差異，但因為這個改動會影響到app端，因此就只好web這邊自己改掉，解決的方法很簡單，就是給所謂的自訂格式

# 自訂格式

如果你有寫過其他語言，可能都會有提供這樣的方法`new Date("2011-10-10 14:48:00", "yyyy-MM-dd HH:mm:ss")`，但是在js中並沒有提供，因此你就必須要自己拆解字串去做到這樣的事情

拆解字串的作法很多種，這邊提供兩種比較常見的

## 字串拆解

固定格式這種的最簡單就是直接拆解字串硬解

```javascript
const dateString = '2019-03-03 10:35:15 +0800';
const date = dateString.substring(0, 10);
const time = dateString.substring(11, 19);
const zoneHour = dateString.substring(20, 23);
const zoneMin = dateString.substring(23);
const result = new Date(`${date}T${time}${zoneHour}:${zoneMin}`);
```

## 使用regex

另外一種作法，就是moment或是luxon的作法，用regex來解析，但在專案中還是有點寫死，不太像是他們是用定義的比較活一點

```javascript
const dateString = '2019-03-03 10:35:15 +0800';
const dateFormat = /^(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) (\+\d{2})(\d{2})$/;
const matchs = dateString.match(dateFormat);
const result = new Date(`${matchs[1]}T${matchs[2]}${matchs[3]}:${matchs[4]}`);
```

# 結論

這個問題的最佳解會是從backend傳回來的資料格式就應該是iso，就不用這麼麻煩的做這些workaround。也許會問為什麼不直接把luxon引入使用就好，雖然`luxon`已經比`moment`還要小，但還是相對用到的功能來說是相對肥阿，不如參考別人的作法，在自己專案中實作比較好

# 參考

[luxon repo](https://github.com/moment/luxon)