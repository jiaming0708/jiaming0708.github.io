---
title: 將專案從 SVN 搬到 Git
date: 2021-12-24 09:35:48
updated: 2021-12-24 09:35:48
categories:
- Git
tags:
- Git
- SVN
---

公司還是有舊的專案是放在 Subversion(SVN) 中，為了接下來增加一些 CI/CD 的輔助，並且希望在 Teams 能夠集中並且即時的收到專案的資訊，就必須要做搬移的動作。

> 之前有做過一次，今天在做的時候又有點卡關，一定是沒有做筆記的原因!! 

<!-- more -->

# SVN 問題

SVN 也有 hooks 可以發通知之類的，公司就有在 hooks 檢查 message 的格式，把 commit 的資訊推到專案管理的平台。但有點美中不足的地方是，hooks 的設定必須進去 server 然後寫 bat 之類的程式，也就是說這個設定對於新建立專案的人來說並不友善。

第二點就是 branch 的管理，每個資料夾只能同步到其中一個 branch ，如果同時有 master 和 develop，就必須要開兩個資料夾來存放，這對於開發人員真的是一個很不方便的事情。(也許可以用一個資料夾做到，如果知道的人可以跟我分享喔)

最後一點應該就是集中式管理，同事會常常要出差到客戶那，或是 VPN 到客戶的環境中作業，這時候將專案放在公司的 SVN server 上，那就會很痛苦要一直切換網路環境。

# 搬移

## 環境準備

電腦應該要安裝好以下這幾個軟體已經被安裝，並且開啟 terminal

* [Subversion](https://subversion.apache.org/packages.html)
* [Git](https://git-for-windows.github.io/)
* [git-svn](https://www.kernel.org/pub/software/scm/git/docs/git-svn.html) (將會透過這個工具做資訊的轉換)

> 作業系統為 window  10

## 取得所有作者資訊

Git 和 SVN 對於作者描述的方法不同，第一步就要先把 SVN 的作者資訊做轉換。

**Subversion users**
![SVN users reference from MS](https://docs.microsoft.com/en-us/azure/devops/repos/git/media/perform-migration-from-svn-to-git/svn-log.png?view=azure-devops)

**Git users**
![Git users reference from MS](https://docs.microsoft.com/en-us/azure/devops/repos/git/media/perform-migration-from-svn-to-git/git-log.png?view=azure-devops)

從 PowerShell (建議使用 version 7) 執行以下的指令，會將作者資訊存到檔案 `authors.txt`，格式會是像 `jiaming = jiaming <jiaming>` 這樣，開啟檔案將 `<>` 中的內容修改成 email 格式。

> 這邊要特別注意一下，檔案的編碼必須是 **UTF8** (不能有 bom)，不然下一個步驟 clone 時會有找不到 author 的錯誤。

```powershell
svn.exe log --quiet | ? { $_ -notlike '-*' } | % { "{0} = {0} <{0}>" -f ($_ -split ' \| ')[1] } | Select-Object -Unique | Out-File -Encoding UTF8 'authors.txt'
```

## 使用 git-svn 複製

接著就要將 svn 的專案轉換成 git，先在目前的專案目錄底下建立一個子目錄 `gitsource` ，並且執行指令。

> 如果只有一個主幹，那直接 `git svn clone` 就好

```shell
git svn clone [svn repo] -A authors.txt ./gitsource
```

若專案是標準的 trunk/branch/tag ，那就可以加上 `--stdlayout` 來描述

```shell
git svn clone [svn repo] -A authors.txt ./gitsource --stdlayout
```

如果 history 很多的話，就會跑比較久一點，建議就是先放著讓他跑，做其他事情去XD

## 轉換 ignore

SVN 也有自己的 ignore 設定，可以透過指令轉換成 .gitignore。

> 如果是 dotnet 可以參考這兩個 ignore 設定，[for core](https://github.com/dotnet/core/blob/main/.gitignore)、[for vs](https://github.com/github/gitignore/blob/main/VisualStudio.gitignore)

```shell
cd gitsource
git svn show-ignore > .gitignore
git add .gitignore
git commit -m 'convert svn:ignore to .gitignore'
```

## 推到遠端

呼~終於到最後一步啦!! 接著就是照平常 push 到遠端的作法，就可以了

```shell
git remote add origin [git repo]
git push -u origin --all
```

# Reference

* [Migrate from Subversion (SVN) to Git - Azure Repos | Microsoft Docs](https://docs.microsoft.com/en-us/azure/devops/repos/git/perform-migration-from-svn-to-git?view=azure-devops)
* [30 天精通 Git 版本控管 (29)：如何將 Subversion 專案匯入到 Git 儲存庫 - iT 邦幫忙::一起幫忙解決難題，拯救 IT 人的一天 (ithome.com.tw)](https://ithelp.ithome.com.tw/articles/10140481)
