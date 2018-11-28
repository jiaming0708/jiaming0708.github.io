---
title: 使用Sandcastle產生文件
date: 2012-10-16 21:49:30
categories:
- Back-end
- C#
tags:
- C#
---
很多人都使用過微軟的Help文件
這種文件是實體檔案，不用透過網路
自己寫的dll產生一個這樣的文件，可以讓其他使用這個dll的人，快速了解如何使用
當然前提是dll有產生XML的註解

<!--more-->

要產生像這樣的文件，需要先準備一些東西
1. Sandcastle
2. Sandcastle Help File Builder，這有GUI可以讓我們快速設定並產生
3. HTML Help Document，產生.chm(HtmlHelp 1.x)檔必備

以上的東西都準備好後，開啟Sandcastle Help File Builder GUI
產生新專案後，可以看到以下的畫面
![GUI](/GUI.jpg)

通常會更改下面的幾個部分
1.FrameworkVersion:dll使用的版本，預設採用電腦中安裝的最大framework version
HelpFileFormat:基本有HtmlHelp1、MSHelp2、MSHelpViewer、Website，這個部分就看需要產生哪種類型的文件囉

2.Help Title:在文件內最上方的Title
HtmlHelpName:文件的名稱

3.PresentationType:基本有hana、Prototype、vs2005、vs2010(似乎是看Builder版本)，個人比較喜歡Prototype

4.這其實才是重點的重點！
Documentation Sources:要產出文件的dll、xml放在這邊
References:產出文件的dll需要的額外參考，就是那種不想產出在文件內，但是沒有又一定會編譯失敗的參考dll

> PS:如果編譯出現「HHC6003 : error : The file Itircl.dll has not been registered correctly」的錯誤
請去下載itcc.dll，並且註冊至系統中regsvr32 [c:/windows/system32/itcc.dll]，[]中是檔案目錄