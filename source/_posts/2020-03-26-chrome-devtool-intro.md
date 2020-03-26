---
title: chrome devtool介紹
date: 2020-03-26 21:06:51
categories:
- Front-end
- chrome
tags:
- chrome
---

今天晚上聽 **林亨力** 在 *GDG台中* 分享的 **如果你連DevTools都不會用，還敢說你是網頁設計者嗎**，滿滿的乾貨，所以趁新鮮趕快來記錄一下自己所不知道的部份。

<!-- more -->

# 命令列選單

在DevTool任何地方按下 `cmd+shift+p` ，就可以叫出選單，才發現滿滿的功能！！

> window請用 `control+shift+p`

![command-list](command-list.png)

## 檢視animation

在命令列選單輸入 `animation` 就只會看到一個選項，點下去就可以在下方看到新的 tab（如下圖）。

![animation](animation.png)

在裡面就能夠看到目前頁面中所有的動畫元素，並且可以暫停重跑。

![animation-result](animation-result.png)

在今天之前，如果要看 animation 的效果並且調整，我會選擇 firefox 查看，現在才知道原來chrome也有類似的功能。
實際操作了一下，發現還是 firefox 比較強，在 firefox 是顯示某個元素底下所有的結果，不用另外開出來，直接在檢視中就提供（如下圖）。

![firefox-animation](firefox-animation.png)

##截圖

想要把整個網頁截取的話，可以不用額外的工具，只需要在命令列中輸入 `screenshot` 選取 `capture full size` 就會開始執行截圖的行為，但需要一下下的執行時間，請耐心等候。

![screenshot](screenshot.png)

## Sensors

一樣在命令列輸入 `sensor` ，這個 tab 中可以看到下面這幾個功能，個人覺得這 tab 的功能很酷！

### Geolocation

可以模擬訊號位置到別的地區，對於這方面的測試是蠻方便的，最簡單的測試是打開 google map 並且變更設定到 `berlin` ，重新整理後就會發現自己離開台灣啦🤩（<s>可以打卡出國了</s>

![geolocation](geolocation.png)

### Orientation

模擬手機的陀螺儀功能，可以到這個[網站](https://intel.github.io/generic-sensor-demos/websensor-panorama/)，並且將 orientation 設定改為 `protrait upsite down` ，重新整理後在右邊手機的地方點著旋轉，就可以上下左右的觀看了🤩

![orientation](orientation.png)

![orientation-result](orientation-result.png)

## Network Condition

在命令列輸入 `network condition` 開啟 tab ，看到下面的 `user-agent` ，可以更改想要模擬的裝置

![network-condition](network-condition.png)

## Coverage

在命令列輸入 `coverage` 開啟 tab 後點擊 refresh 的圖示，就可以看到你網站的所有 css 以及 js 檔案使用率，對於效能調教上有著很大的幫助。基本上應該是越少紅色越好，因為紅色代表著未被該頁面使用到。

![coverage](coverage.png)

# 偵錯

## design mode

在 console 的 tab 中輸入 `document.designMode='on'` 就可以直接在畫面上編輯元素的文字，這對於要測試多文字樣式的確認還蠻方便的。

# CSS overview

目前在 `stable` 版本是沒有的，可以看看[別人的介紹](https://umaar.com/dev-tips/209-css-overview/)，會讓我有點期待想要看到公司的網站會顯示什麼樣的內容

# Reference

* [slide](https://docs.google.com/presentation/d/e/2PACX-1vQhCWJLiTCLcEhPsL1-ixogj6vTqzv4FW5KKXb6UmxCE87Omr1CctZyXXIb4igmD1B1cdqPPacfTeN2/pub?slide=id.g71b87befdc_0_6)