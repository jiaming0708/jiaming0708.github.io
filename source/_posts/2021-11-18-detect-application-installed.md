---
title: '在瀏覽器偵測裝置是否已經安裝應用程式'
date: 2021-11-18 10:10:54
updated: 2021-11-18 10:10:54
categories:
- Frontend
- JavaScript
tags:
- JavaScript
---

前面介紹了在瀏覽器開起應用程式及封裝應用程式，今天要來介紹的是如何在瀏覽器中偵測裝置是否已經安裝，根據狀態來決定是否要顯示不同的文字內容。

> 還沒有看的話，可以先回頭看看這兩篇 {% post_link browser-open-application %}、{% post_link vs-installer-project %}

瀏覽器開啓應用程式是透過os層的機制處理，對於瀏覽器來說，不會回報是否開啓成功，這個部分就必須要透過其他的手法來偵測。

<!-- more -->

# 實作

要偵測是否開啟成功的話，就不能開在新視窗或分頁，也不能把現在頁面給換掉，這樣 script 都會失效。這邊可以透過 iframe 的方式來做包裝，將 custom protocol 丟進去，接下來偵測的行為才能夠正確。

> iframe 這時候就是香阿，在其他地方使用還是要小心XD

如果有安裝的話，會跳出視窗詢問是否開啟應用程式，對於原本的頁面來算是 `unfocus/blur` ，大部分的瀏覽器可以透過這樣的概念來做到偵測的行為。
接著來看一下基本的範例，利用 **blur** 的事件來判斷是否開啟，並用 **timeout** 來判斷是否沒安裝。

> 這邊要注意到 timeout 的時間，可能會因為電腦的速度而影響到

```javascript
// 利用 timeout 來判斷是否有安裝
const timeout = setTimeout(function() {
    console.log("fail to launch application");
    handler.remove();
  }, 2000);

var iframe = document.querySelector("#hiddenIframe");
if(!iframe) {
  document.createElement("iframe");
  iframe.src = "about:blank";
  iframe.id = "hiddenIframe";
  iframe.style.display = "none";
  document.appendChild(iframe);
}
// 若離開視窗，則表示有安裝
const onBlur = () => {
  clearTimeout(timeout);
  handler.remove();
  console.log("launch application successful");
};
const handler = registerEvent(window, "blur", onBlur);

iframe.contentWindow.location.href = "jimmy://";
```

# 結論

實作的概念大概是這樣，其實已經有人寫好並且包成 library 可以直接引用，想要看完整的作法就請移駕到 [Custom Protocol Check in Browser](https://github.com/vireshshah/custom-protocol-check) 。

這個作法也適用於手機上，判斷若沒有安裝的話就提示安裝 app 這樣的概念，之前想要用的時候沒查對方向，變成是用手機開啟都會提示安裝 app ，但這樣其實蠻干擾使用者的，透過這樣的機制，能夠對使用者更加友善。
