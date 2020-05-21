---
title: 使用 Github Action 作為 CI/CD 將 libraray 發佈到 Github Package
date: 2020-05-21 21:43:19
categories:
- CI/CD
tags:
- Github
- CI/CD
---

Continuous Integration (以下簡稱CI)，是在開發中很重要的一環，大家常用或常聽到的例如：Jenkins, Travis...等等的，我們常在使用的 Github 本身也有提供類似的服務。
今天來使用 Github 的全套服務，使用 Action 做 CI/CD 並且發佈到 Package。

<!-- more -->

# 使用Action

> Github本身對於Action有一些[限制](https://help.github.com/en/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions)和[收費](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#usage-limits)的方式，詳細請參考官方文件。

## 使用範本建立

在開始之前當然就要先建立好一個 repo ，點選到 Action 的頁籤，就可以開始建立囉

![start action](start-action.png)

一個還沒有建立過 Action 的專案，會先看到上圖的畫面，可以從現有的範本中做建立的動作，那我們就使用 `Simple workflow` 來做為我們的第一個 Action，從範本中我們可以發現到兩件事情

* `yaml` 的格式做撰寫
* 檔案存在 `.github/workflow` 底下

![simple action](simple-action.png)

先什麼都不改然後 commit 就可以到 Action 的頁籤中確認執行狀況，從下圖中可以看到 `CI` 的 workflow 已經執行成功

![commit](commit.png)

## 了解yaml

為了之後能夠寫出自己的檔案，就要先了解檔案的每個區塊是怎麼運作，詳細可以[參考官網](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions)

> 為了篇幅我將註解都刪除

```yml
name: CI
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Run a one-line script
      run: echo Hello, world!
    - name: Run a multi-line script
      run: |
        echo Add other actions to build,
        echo test, and deploy your project.
```

### name

workflow 的名稱，為了區別不同的行為，可以將名稱取成不一樣。

### on

觸發 workflow 的時機點，按照範例觸發時機是 push 新 commit 或是對這個 branch 發 PR 。

### jobs

定義要執行的工作項目，可以一個或多個*（預設每個工作項目是平行處理）*。

#### build

第一層的 **build** 是 job_id ，若有多個 job 請記得不要重複並且取好懂一點的名字

#### runs-on

要以哪個環境做執行 (window/ubuntu/macos)，支援列表可以[參考這邊](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#github-hosted-runners)

#### steps

執行的步驟，會按照順序的往下執行

##### uses

使用別人已經寫好的功能作為基礎來進行後續的動作，以此範例是用來 checkout repo

> 官方建議後面要加上版本號，避免突然間炸掉

##### name/run

每個步驟的名稱，以及要執行的指令

## 看看build log

![build log](build-log.png)

可以看到左邊的 **build** 就是所有的 job，右邊則是列出所有的 steps

###執行錯誤

![build fail](build-fail.png)

如果有執行錯誤從外面其實看不到是在哪一個步驟錯誤，必須要到詳細的 log 中查看才行

# Package

前面已經了解完了 Action，接著第二個目的是使用 Github Package 這個服務，這個服務本身不是只能放 javascript 的 library 也是能夠放其他語言的 library ，例如 nugget、gem 等等的，詳細請見[官方說明](https://github.com/features/packages)

## 登入

使用 npm 的方式做登入，只是要另外指定 registry；登入的時候使用的是 Github 的帳號以及 **token** ，token產生方式可以[參考這邊](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)

> 因為我們要用的是 package 功能，所以除了 repo 以後還要將 package 的權限也打開

```shell
$ npm login --registry=https://npm.pkg.github.com
> Username: USERNAME
> Password: TOKEN
> Email: PUBLIC-EMAIL-ADDRESS
```

## 設定 package.json

要發佈到 package 之前要先將 package.json 設定好，這邊有兩個地方要注意一下

* publishConfig 要設定 registry
* name 必須是 `organization`/`personal account` 加上這個 repo 的名稱

> 我們的 repo 假設叫做 `jimmy-first-library` 那我們發佈到 npm 的時候可以改名稱叫做 `jimmy-library`，但是在 Github 是不被允許的，因為他是基於 repo 去產生的 package 所以名稱必須相同

```json
{
  "name": "@jiaming0708/schematics-react",
  // ...
  "publishConfig": { "registry": "https://npm.pkg.github.com/" }
}
```

## 發佈

做完以上的動作以後，我們就可以用 npm 的 publish 指令，進行發佈

```shell
npm publish
```

接著我們可以到 Github 的 repo 中查看

![repo package](repo-package.png)

![package info](package-info.png)

## 安裝

發佈以後我們可以來安裝看看，在專案的根目錄中，加上 `.npmrc` 在裡面要加上 package 所屬的 owner，接著就可以使用一般的 npm 安裝方法

```shell
registry=https://npm.pkg.github.com/jiaming0708
```

> 如果你是到另一個環境做測試的話，記得要重新登入

# 使用 Action 發佈到 Package

我們要在steps底下，加上下面的語法

```yml
    - name: Setup Node.js environment
      uses: actions/setup-node@v1
      with:
        node-version: '12.6.0'
        registry-url: 'https://npm.pkg.github.com'
        scope: '@jiaming0708'
    - run: npm publish
      env:
        NODE_AUTH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

## node環境建置

使用別人寫好的 node 環境，並且加上 `with` 指定一些參數

## 發佈

執行發佈的時候，一樣要提供 token 讓 action 來幫我們登入，這邊必須要將 token 放在 env 的設定中

這邊有一個很厲害的地方， `GITHUB_TOKEN` 是一個預設的變數，每次會執行時自動產生並且提供對應的權限，因此你不需要特別的再產生什麼樣的 token 來給 action 使用

# 總結

Github Action 基本上蠻方便的，除了做 CI/CD 以外，也可以拿來做 issue 上的 tag，對於一個很多人在使用的 open repo 管理上會很有幫助。 

Github Package 以 javascript 就是另一個私人的 libraies，使用上較為麻煩，一定要做登入的動作； repo 和 library 名稱一定要相同，不然會發佈失敗。

第一次的使用上來說對於 Action 的評價高過於 Package，但因為 npm 已經被 github 買下，所以未來會不會移到 package 也不好說。

# reference

* [sample repo](https://github.com/jiaming0708/schematics-react)
* [Learn YAML](https://www.codeproject.com/Articles/1214409/Learn-YAML-in-five-minutes)
* [Workflow syntax for GitHub Actions](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions)
* [Github package documentation](https://help.github.com/en/packages)