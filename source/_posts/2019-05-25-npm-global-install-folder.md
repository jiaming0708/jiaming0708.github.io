---
title: '[抓蟲] npm全域安裝路徑錯誤'
date: 2019-05-25 17:20:19
updated: 2019-05-25 17:20:19
categories:
tags:
---

很久沒寫angular，最近接了一個小的靜態網站，想要拿牛刀來殺雞XDDD
第一件事情當然就是更新最新的angular cli，但是更新後系統卻跟我說 `ng command not found`

<!--more-->

# 問題情境

先來看看電腦環境

> macos一開始直接裝node後來改裝nvm
> node: v9.3.0

既然我們是用nvm那我們安裝的npm global應該是跟著nvm底下跑，但當我下指令 `npm i -g @angular/cli` 卻一直跑去另一個node version的目錄底下

```shell
/Users/jimmy/.nvm/versions/node/v6.11.1/bin/ng -> /Users/jimmy/.nvm/versions/node/v6.11.1/lib/node_modules/@angular/cli/bin/ng
```

因此這樣在目前版本的node目錄中永遠不會有`@angular/cli`可以使用，但是到底是哪邊的設定影響到呢？

# 問題排查

## nvm版本

第一次先確認的是nvm中的node版本是不是根本就錯誤，因此先把`v6.11.1`以及`v9.3.0`版本刪掉，接著重新安裝`v9.3.0`並且設定為預設

```shell
nvm uninstall v9.3.0
nvm uninstall v6.11.1
nvm install v9.3.0
nvm alias default v9.3.0
```

再次安裝cli但還是失敗

## 重新安裝nvm

因此改成檢查nvm本身是不是正確的，可以參考一下這篇文章[Node.js 環境設定-for mac](<https://medium.com/@toumasaya/node-js-%E7%92%B0%E5%A2%83%E8%A8%AD%E5%AE%9A-for-mac-a2628836feaf>)，但我們按照步驟重新做完以後，再次安裝cli還是一樣的結果

## npm

最後試著往npm的相關設定找，很剛好的讓我找到了`.npmrc`在根目錄底下有設定了一個`prefix`的目錄就是`v6.11.1`，先註解掉試試看，再次安裝cli果然成功，可以找到**ng**的指令了

# npmrc

試著回想一下為什麼會設定prefix...原來是因為之前vscode的terminal都會一直說prefix有問題，因此找到了一個錯誤的設定，但也剛好能夠解決我的問題，就這樣用下去，設定完以後也一直沒有再使用其他的global套件，因此相安無事到現在orz

但我們還是要來了解一下`.npmrc`，參考一下[官方的文件](<https://docs.npmjs.com/files/npmrc>)

prefix的目的是什麼呢，npm的指令全部都會改成到prefix指定的路徑下，而不是目前的node目錄，因此這個如果設定在`~/.npmrc`就會發生像我一樣的悲劇

# 結論

要解決問題也要找對資料，不然就可能造成悲劇，其實vscode會報那個錯誤是因為當初我直接安裝node變成用nvm，因此一些設定上錯誤，這次找到[其他資料](<https://medium.com/@mvpdw06/%E8%A7%A3%E6%B1%BA-nvm-is-not-compatible-with-the-npm-config-prefix-option-currently-set-to-usr-local-%E5%95%8F%E9%A1%8C-cb1f3462ef40>)才發現原來問題是在用brew安裝nvm，最後成功解決問題，可以繼續使用angular啦！！