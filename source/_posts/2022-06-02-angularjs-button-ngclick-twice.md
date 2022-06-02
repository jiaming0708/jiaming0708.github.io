---
title: '[除蟲] AngularJS button click 執行兩次'
date: 2022-06-02 14:39:44
categories:
- Front-end
- AngularJS
tags:
- AngularJS
- Debug
---

同事詢問了一個 AngularJS button click 的時候會執行兩次的問題，只有在 IE 上會發生，但在 edge/chrome 就不會。聽到這個問題，第一個反應是先去看看網路上有沒有人有類似的問題，`angularjs button click execute twice` ，果然我們不是孤單的，蠻多人討論過。

> 比較多人提到的，反而是在 safari 上會有這個行為。

<!-- more -->

- [ngClick using ngTouch fires twice · Issue #6251 · angular/angular.js (github.com)](https://github.com/angular/angular.js/issues/6251)
- [javascript - AngularJS ng-click fires twice - Stack Overflow](https://stackoverflow.com/questions/18709297/angularjs-ng-click-fires-twice)
- [javascript - Why are multiple click events fired when using ngTouch? - Stack Overflow](https://stackoverflow.com/questions/21708730/why-are-multiple-click-events-fired-when-using-ngtouch)

看到兩個方法可以解決這個問題

- button 增加屬性 `type="button"` (最後選擇這個解法)

[MDN](https://developer.mozilla.org/zh-TW/docs/Web/HTML/Element/button) 有提到 button 的 type 預設值是 `submit`，通常 submit 會對應到 form 的 post 行為給 API，在我們的程式中並沒有被包在 form 裡面。
找到黑暗大也曾經抓過類似的問題，[【茶包射手日記】當IE遇上Enter-黑暗執行緒 (darkthread.net)](https://blog.darkthread.net/blog/enter-submit-in-ie/)，提到 submit 如果沒有 form 的話會觸發 click 事件。

> 不管是不是 IE 或是什麼的，讓元素有正確的行為，其實是撰寫的人應該要做到的。

- `event.preventDefault(); event.stopPropagation();`

`preventDefault` 或者是 `stopPropagation` 就很單純了，避免[事件的冒泡](https://blog.techbridge.cc/2017/07/15/javascript-event-propagation/)，當然就可以解決掉重複執行的問題。

回過頭來看 angularjs 自己如何針對這個 issue 做修正，其中來看幾個原本已經改好但最後關閉的 PR

- [fix(ngTouch): remove ngClick override by Narretz · Pull Request #13852 · angular/angular.js (github.com)](https://github.com/angular/angular.js/pull/13852/files)
- [fix(ngTouch): deprecate ngClick override directive, disable it by def… by Narretz · Pull Request #13857 · angular/angular.js (github.com)](https://github.com/angular/angular.js/pull/13857/files)

上面兩個都是針對 ngTouch 做修改，其中還有把引入 ngClick 的檔案移除掉 (個人是蠻懷疑這是造成 submit 會重複送的原因)，最後這兩個 PR 直接被重新的調整並且放入到 1.5 的版本 [AngularJS: Developer Guide: Migrating from Previous Versions](https://docs.angularjs.org/guide/migration#ngtouch-ngclick-)。