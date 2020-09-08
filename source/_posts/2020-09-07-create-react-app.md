---
title: create-react-app 開發心得分享
date: 2020-09-07 21:11:31
categories:
- Front-end
- React
tags:
- React
---

最近公司的一個新專案，因為網站的功能都必須要登入以後才能操作，因此選擇了 `create-react-app (CRA)` 這個來做為 project base，前陣子剛上線第一個版本，順便來整理一些心得。

<!-- more -->

# why CRA

在開始之前先和 PM 確認這個網站的功能要有哪些，是否會有所謂的內容或是行銷頁面，那經過 PM 確認後，網站主要是登入後的內容。既然要登入，有沒有所謂的 server-side-render (SSR) 根本不重要，因為對於網站來說也沒辦法那些內容呈現給爬蟲，因此這個時候我們就只需要 client-side-render (CSR) 的功能。

既然只是 CSR 的網站，react 的團隊也提供已經建置好的一個架構也就是 CRA，讓你不用再煩惱 webpack 等等的設定。

# 專案建置

CRA 本身提供的功能真的很陽春，react 團隊只是提供了一個平台，讓你不用在乎專案的一些架構設定等等，但如果要達到團隊開發需求的話，還是有不少東西需要被調整，接著我會說明怎麼去建置團隊所需要的環境。

## 建立有redux的專案

根據[官網的介紹](https://create-react-app.dev/)，我們可以使用下面的指令很輕鬆的建立好專案，完成後使用 vscode 打開，可以發現到只有 `public` 以及 `src` 兩個目錄而已，而且是只有 `react` 的功能， `redux` 等等的功能都沒有。

```shell
npx create-react-app my-project
cd my-project
code .
```

作為一個完整的專案，基本上就是會有 `redux` 以及 `middleware` （也許有人會說用 context 就好，但就不再這邊討論囉！），那有沒有更簡單的方法呢？當然是有的， 在 npm 中搜尋 [cra-template](https://www.npmjs.com/search?q=cra-template-*) 就可以使用這些別人已經建立好的範本來做初始專案。

目前團隊是使用 redux+saga ，因此我這邊就選擇一個已經整合好的[範本](https://www.npmjs.com/package/cra-template-react-redux-saga)。

```shell
npx create-react-app my-project --template react-redux-saga
cd my-project
code .
```

## CSS管理

在 CRA 中有兩種的 css 管理方法，一種是全域的檔案，另一種就是 css module ，基本上兩種模式都有用到，但主要用比較多的會是 css module ，因此再建立 component 時，就必須要將 css 的檔案名稱加上 ***.module.css** 這樣才會變成 module scope，不然的話預設就會是全域。

另外就是使用預處理器 scss 來撰寫，webpack的處理已經準備好，只是需要安裝 `node-sass` 的 library 即可！

```shell
yarn add node-sass
```

## Route

建立一個稍微有點規模的專案，都一定不會只有一個頁面，因此我們一定會需要使用到 route ，那想當爾就是我們要自己安裝囉！

```
yarn add react-router-dom
```



## Format 及 ESLint / StyleLint

排版是一件麻煩的事情，尤其是大家的習慣不同，使用 formatter 以及 linter 是最簡單的，我們團隊是使用 [prettier](https://github.com/prettier/prettier) 為基礎，另外套一些 react / scss 的 lint，基本上不太會去改什麼設定，[參考這邊使用方法](https://create-react-app.dev/docs/setting-up-your-editor/#formatting-code-automatically)。

如果有另外定義 `.eslintrc` 記得要再 env 檔案中加上 `EXTEND_ESLINT=TRUE`，才不會一直套用到 CRA 預設的 eslint

> 使用 ESLint 時要注意一下，如果開啟 error 會導致編譯失敗，一定要修復才會正常編譯（但我還是開了！）

## 環境變數

CRA 已經定義好每個 script 所對應的 env 檔案是什麼，先參考一下[這邊](https://create-react-app.dev/docs/adding-custom-environment-variables#what-other-env-files-can-be-used)。公司的開發環境一共有三個，分別是**本機**、**測試**、**正式**，但根據文件所介紹的， build 只能對應 `.env.production` ，那測試環境的 `.env.uat` 檔案該怎麼讓 build 的指令可以被使用到？還好他們也有考慮到這問題，可以參考[這邊](https://create-react-app.dev/docs/deployment#customizing-environment-variables-for-arbitrary-build-environments)，透過 `env-cmd` 的 library來替換檔案。

# 結論

其實可以發現 CRA 真的很基本，著重在於編譯相關的事情，但也有考慮到一個專案應該會要有的東西，只是需要使用者自己去針對所需做設定，也算是蠻符合 react 本身的精神。

> 原本還想寫寫客製 `react-scripts` ，但沒有想出比較好的介紹方式，這邊就先不提了，有興趣的可以從 `eject` 的相關介紹著手。

# Reference

* [CRA 官網](https://create-react-app.dev/)