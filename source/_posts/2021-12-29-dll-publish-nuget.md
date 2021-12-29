---
title: 將獨立的 dll 打包發佈到 nuget server
date: 2021-12-29 09:02:29
categories:
- nuget
tags:
- nuget
---

nuget 是 dotnet 的套件管理工具，就像是 npm 是屬於 nodejs 的平台一樣，在上面的版本只能升不能降，還會有所有的版本異動紀錄，可以讓專案的參考管理更為有效。
有 source code 的話要做到打包其實非常的簡單，今天要分享只有 dll 的話該如何發佈到 nuget server。

<!-- more -->

# 安裝 Explorer

要發佈到 nuget 前要先做打包成 `*.nupkg` 的檔案，有 source code 的話可以透過 `nuget.exe` CLI (下載 [NuGet Gallery | Downloads](https://www.nuget.org/downloads)) 來進行打包，打包後如果還要編輯 unpkg 的話，就可以透過 [Nuget Package Explorer](https://github.com/NuGetPackageExplorer/NuGetPackageExplorer) 。

目前有兩個方法可以安裝

* [Microsoft Store](https://www.microsoft.com/store/apps/9wzdncrdmdm3) (推薦)
  ![microsoft store].\microsoft-store.png)

* [Chocolatey](https://chocolatey.org/packages/NugetPackageExplorer)

  ```shell
  choco install nugetpackageexplorer
  ```

# 打包 dll

打開 `NuGet Package Explorer` 並且選擇 `Create a new package` 來產生一個新的檔案

![create new package](./create-new-package.png)

接著會看到畫面分為兩個區塊

* metadata: 描述 package 的基本資訊，版本、作者、說明...等。
* contents: 包含在這個 package 中的內容，dotnet 版本、dll...等。

![package view](./package-view.png)

## 編輯 metadata

點下 metadata 的左上角的 icon，進入編輯的畫面

![edit metadata](./edit-metadata.png)

通常會編輯幾個區塊

* Id: 需要唯一名稱，不可和 nuget 平台上的重複 (同一個 Id 可以在不同的 nuget server 上，也就是 nuget.org 和 private 的可以用同一個名稱)。
* Version: 每次推到 nuget server 都必須比原本的版本還要新。
* Author: 作者資訊。
* Description: 對於這個 package 的描述。

## 編輯 content

要將 dll 放進去前，要先新增 `lib` 的資料夾

![add lib folder](./add-lib-folder.png)

如果 dll 有指定平台或是版本，就可以選擇上面的 `Add {*} folder` ，若沒有的話，可以直接點選 `Add Existing File` 。
我要放進去的 dll 是 framework 4.5 的版本，就先指定版本再來新增檔案。

![specify dotnet version](./specify-dotnet-version.png)

![add dll in content](./add-dll-in-content.png)

## 產生 unpkg

做完前面兩個步驟以後，就可以按下儲存，來產生 unpkg 的檔案。預設的檔案名稱會使用 `{Id}_{Version}.unpkg` ，基本上也不用去改名稱，直接找到想要的目錄底下儲存就好。

![save unpkg](./save-unpkg.png)

## 發佈

最後就是將做好的內容發佈到 nuget 上啦，在上方的選單中 `File` 點選 `Publish` ，選擇要發佈的 server 及填入 api key。

![publish](./publish.png)

# 心得

對於一些年久失修的 dll，可以透過這樣的方法來重新打包上 nuget server，讓套件的管理更加統一。一開始操作的時候，我也有嘗試過將好幾顆 dll 一起打包，但這樣會有個缺點，一次參考就一定會所有 dll 都進去，如果每個 dll 是獨立沒有相依性的，就會建議拆開來做，避免這樣的問題發生。這些做好的 unpkg 甚至可以放進版控並且整合 CI/CD 流程。

# Reference

* [NuGetPackageExplorer/NuGetPackageExplorer: Create, update and deploy Nuget Packages with a GUI (github.com)](https://github.com/NuGetPackageExplorer/NuGetPackageExplorer)
* [[Nuget\] 使用 NuGet Package Explorer 製作 Nuget 套件 ~ m@rcus 學習筆記 (marcus116.blogspot.com)](https://marcus116.blogspot.com/2019/02/nuget-nuget-package-explorer-nuget.html)