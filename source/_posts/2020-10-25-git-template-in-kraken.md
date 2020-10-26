---
title: 在GitKraken中使用template
date: 2020-10-25 15:33:08
categories:
- Git
tags:
- Git
- GitKraken
---

最近團隊開始規範 git commit 並且導入 [Conventional commit](https://www.conventionalcommits.org/en/v1.0.0/)，剛開始不熟悉都會一直要去翻文件，看看到底這時候要用什麼樣的類型，為了加速類別的輸入，可以在 commit 的 template 中設定，今天就來分享一下使用這個規範並且如何有效的套用在 [GitKraken](https://www.gitkraken.com/) 這套工具中。

<!-- more -->

# 介紹Conventional commit

這是一個由 angular 團隊推動的 commit message 規範，主要是把每個 commit 做一個分類，分類的好處除了人在看比較輕鬆以外，也是有機會能夠使用工具去拿到某個分類的全部 commit，在 release 時要整理就會比較輕鬆。

目前的類別大概有 `feat`, `fix`, `build:`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf`, `test`...等，基本上大部分的 commit 應該都能夠被分類到；目前我最常用到的應該是 `feat`, `style`, `refactor`, `fix` 這幾個，剩下幾個就沒那麼常使用到。

要導入一個規範最好的方法還是用強迫的，那就必須要安裝 commit lint 的套件 [config-conventional](https://github.com/conventional-changelog/commitlint/tree/master/%40commitlint/config-conventional)。

```shell
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```

# 使用Git Template

剛剛有說到可以使用套件來作為規範，但初期的時候就是很容易忘記有哪些類別，這時候可以考慮使用 git 的 template 功能，讓我們在 commit 時，能有一個快速的參考，當然這個 template 並不會真的進入到你的 message 中。

接著就來試試看在 git 中加上 template，首先建立一個檔案叫做 `.gitmessage` ，並且放在 root 底下。

```shell
cd ~
touch .gitmessage
vi .gitmessage
```

接著將 conventional commit 的類別放進去，輸入 `i` 然後 `cmd+v` 貼上，這個是我目前使用的內容。

```
# Conventional Commit Types:
# build
# ci
# chore
# docs
# feat
# fix
# perf
# refactor
# revert
# style
# test
```

最後把這個檔案指定給 git。

```shell
git config commit.template .gitmessage
```

# 設定 GitKaren

> 如果你是 CLI 派的，就麻煩這邊左轉離開囉。

使用過的 GUI 工具還蠻多種的，目前就剩下一套 GitKaren 固定在使用，關於這套的介紹建議大家可以去參考 [Mike 的文章](https://wellwind.idv.tw/blog/2018/04/03/git-using-gitkraken-1-basic/)，畢竟介紹這套工具不是我們今天的重點，今天的重點在於使用了 commit template 以後，怎麼樣更方便的在 GitKaren 中應用。

做完上面的步驟以後，就可以看到在 message 的欄位已經自動帶入所建立的 template。

![message template](message_template.png)

這時候可以先來試一下 commit 一個檔案，看看結果是什麼。

![try a commit](try_a_commit.png)

唉～等等！為什麼我們的 template 也跟著進去，這跟我們預期的不同，如果你有 `SourceTree` 這套 GUI 的話也能夠試試看，結果應該會有點不同。

讓我們試著找找看設定沒有沒地方可以修改，開啟 `preference` 可以看到下方有一個屬於個別專案的設定，點進去以後將 `remove comment from commit messages` 打勾即可。

> 這個地方比較麻煩，還沒找到好的設定可以套用全部，如果你有找到方法，歡迎留言告訴我

![preference for repo](preference_for_repo.png)

![remove comment](remove_comment.png)

# 參考

* [Conventional commit](https://www.conventionalcommits.org/en/v1.0.0/)
* [GitKraken offical site](https://www.gitkraken.com/)
* [GitKraken commit template](https://support.gitkraken.com/working-with-commits/commits/#commit-templates)

