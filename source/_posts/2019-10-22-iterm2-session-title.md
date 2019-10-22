---
title: '[Debug] iterm2的頁籤標題設定'
date: 2019-10-22 21:47:48
categories:
- iterm
tags:
- iterm
- Debug
---

一直以來都有用 `iterm2` 作為bash的工具，最近不知道什麼樣的更新，讓tab的標題永遠都只會顯示 `zsh` ，但工作的關係，一次都會開好幾個專案在跑，那每個都顯示 `zsh` 說真的還真沒記得哪個tab是什麼...

<!-- more -->

![incorrect_title](incorrect_title.png)

上圖是壞掉的狀況，為了示範只有兩個 tab，但是實際上在工作的時候我會開四五個，到底是哪邊的設定跑掉呢？

首先，要先定義一下名詞，在 `iterm` 裡面，每個 tab 其實叫做 session ，這樣在查找資料的時候才會比較容易，不然會像我一樣一直用 tab title 去查，查到的不一定是錯的，但就是解不了我的問題

# 設定

這個問題其實很簡單，只是一個設定跑掉而已，找了很久才找到，打開 `preferences` 的設定，進入到 `Profiles` 的頁籤，然後找到 `Title` 這個下拉選單

## 錯誤

![profile_title_incorrect](profile_title_incorrect.png)

從上面這張可以看到，只選了 `Job` 然後其他都沒了，試著每個都選選看可以看到不一樣的變化

## 預設

那預設的是什麼呢？應該要選 `Session Name + Job`

![profile_title_correct](profile_title_correct.png)

設定完的 `iterm` 就會回到正常的資料夾顯示於標題

![correct_title](correct_title.png)

# 參考

[https://iterm2.com/documentation-session-title.html](https://iterm2.com/documentation-session-title.html)