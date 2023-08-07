---
title: Remix 佈署到 Github Pages
date: 2023-08-07 20:47:36
updated: 2023-08-07 20:47:36
categories:
- CI/CD
tags:
- CI/CD
- Github Action
- Github Pages
- Remix
thumbnail:
---

使用像是 [Remix](https://remix.run) 所開發出來的網站，有沒有可能直接架設在 Github Pages

> 答案是有機會的

<!-- more -->

要先知道 Github Pages 只提供靜態網站的服務，如果網站一定要 nodejs 才能跑起來的，就不算是靜態囉。

> 謎之音：那不就沒戲唱了？

搞清楚 Github Pages 可以提供的服務後，再來確認自己在 Remix 上開發什麼樣的功能

- 都是前端功能，用 SSR 來提高 performance
- 也有 API 功能

像是 Remix 官方的教學，都是有含資料庫及自己也有 API 的功能，這種就沒辦法架在 Github Pages。
那也就是說只要在 Remix 只有寫一些前端的部份，沒有 API 也沒有資料庫，就有機會把這個網站直接架在 Github Pages 上。

主要是透過 linux 的指令 [wget](https://www.systutorials.com/docs/linux/man/1-wget/) 來幫助我們完成，底下是指令的功能敘述

> GNU Wget is a free utility for non-interactive download of files from the Web

如果只有前端的功能，也就是說當我們把 Remix 跑起來後就可以去下載整個網站會用到的檔案，並且把這些檔案放到 Github Pages 上面。
等等會用底下的指令來做下載及佈署。

```shell
wget --mirror http://localhost:3000 -P public --no-host-directories --page-requisites --adjust-extension
```

那就來透過 Github Action 做佈署，把網站架在 Github Pages 上吧。

```yaml
name: deploy

on:
  push:
    branches: [ main ]

jobs:
  build:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [20.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
    - uses: actions/checkout@v2
    - uses: n1hility/cancel-previous-runs@v3
      with: 
        token: ${{ secrets.GITHUB_TOKEN }}

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v2
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm run build --if-present

    - name: Start server and mirror it with wget
      run: |
            npm start &
            sleep 10 &&
            wget --mirror http://localhost:3000 -P public --no-host-directories --page-requisites --adjust-extension

    - name: Deploy to GitHub Pages
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./public
```

這個方法其實算是偷吃步，不過會用 Github Pages 來架設的網站本身應該也不會太複雜，所以簡單用用也是不錯的。

## Reference

- [wget (1) - Linux Manuals](https://www.systutorials.com/docs/linux/man/1-wget/)
- [Create a static site with Remix and wget and deploy it to GitHub pages](https://blog.oldweb2.com/remix-static-site)
- [Remix](https://remix.run/docs/en/1.19.2)