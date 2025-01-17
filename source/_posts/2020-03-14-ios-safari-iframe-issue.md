---
title: '[CSS] ios的iframe問題'
date: 2020-03-14 16:22:46
updated: 2020-03-14 16:22:46
categories:
- Frontend
- CSS
tags:
- CSS
- debug
---

如果要掛載另一個網站的內容進來時，通常會使用iframe，今天要來分享一下在ios safari的bug，並且要如何解決

> iframe使用雖然方便但也要小心一些安全性的問題

<!-- more -->

# 問題

發現的問題有兩個，也都蠻容易重現的，剛好有用到iframe的話，就必須要注意一下！

* 寬度超過100%
* 捲動穿透看到iframe後層的內容

![issue](issue.gif)

可以從底下的button看到，寬度是不正確的，並且在捲動的時候，有個莫名的背景出現。

>  發生環境是在 **ios 11 & 12** ，在 **ios 13** 已經修正。
> （更低版本的就沒有特別測試，因為公司產品的 app 最低也只有支援到 11）

# 解決方法

先來看一下html的結構

```html
<body>
	<div>原始網站內容</div>
	<div style="position: absolute;right: 10px;bottom: 10px;width: 300px;max-width: 100%;">
		<iframe style="width: 100%;height: 100%"></iframe>
	</div>
</body>
```

## 寬度超過100%

首先看一下 iframe 的父層設定 `width: 300px;` ，為了讓 iframe 佔滿整個空間，因此設定 `width: 100%;` 但是在safari上偏偏就是會發現他超過了！！

必須要使用一個workaround來解決這個問題，不要直接設定 `width` 而是改成設定 `min-width` 並且加上 `scrolling` 的屬性。

```html
<iframe style="width: 0;min-width: 100%;" scrolling="no"></iframe>
```

如果同時需要支援手機和桌機的話，記得加上檢查瀏覽器版本的語法來控制 `scrolling` 的屬性

```javascript
function isiOS(){
	return /iP(hone|od|ad)/.test(navigator.platform);
}
```

## 捲動穿透

捲動的問題可以透過 `overflow` 相關的兩個屬性來解決，這次就不是放在 iframe 身上，而是要放在父層才會有效果；目前測試起來不會因為手機或桌機而有所區別。

```html
<div style="-webkit-overflow-scrolling: touch;overflow-y: scroll;">
  <iframe></iframe>
</div>
```

# 結論

這些問題在safari似乎存在很久，使用關鍵字 **ios iframe issue** 蠻容易查到相關的資料，但不知道為什麼 apple 拖到 `ios 13` 才修正掉，只能說前端真的很辛苦，要面對各種瀏覽器廠商所留下的坑。

順帶一提，如果是用mac的朋友，可以使用 `simulator` 來進行測試，還能透過本機的safari來針對device上的做檢視。

> safari真的是新一代的IE

