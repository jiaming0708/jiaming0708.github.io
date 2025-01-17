---
title: 安裝 PowerShell7 及 Window Terminal 並套用 theme 美化
date: 2021-05-19 21:09:59
updated: 2021-05-19 21:09:59
categories:
- Windows
tags:
- Windows
- Powershell
- WindowTerminal
---

最近再次投入了 window 平台的筆電，之前已經很習慣 mac 的 iTerm + oh-my-zsh 的功能，一直嘗試找有沒有類似的工具可以使用，很開心的找到了一個類似的工具，[Window Terminal](https://github.com/microsoft/terminal) 搭配最新的 [PowerShell 7](https://github.com/PowerShell/PowerShell) 套用 [oh-my-posh](https://ohmyposh.dev/) （是不是很相似阿，作者有說是參考 oh-my-zsh）

<!-- more -->

# 安裝 Window Terminal 以及 PowerShell 7

## PowerShell 7

PowerShell 7 或者是說 PowerShell Core，比原本 Window 10 內建的 PowerShell 5.1 相比，更加強大且跨平台。這套在安裝後會獨立存在，也就是說不會蓋掉原本的 PowerShell 5.1，也就是說會有兩套同時存在於電腦中。

> 我是蠻懷疑，有多少 Linux 或是 Mac 的使用者會想要安裝這套XDDD

現在微軟都有提供多種安裝方式，可以到 [github](https://github.com/PowerShell/PowerShell) 看看，那個人是使用 [Window Store](https://www.microsoft.com/store/productId/9MZ1SNWT0N5D) 直接下載安裝，安裝完以後可以在**開始**的選單中搜尋 `powershell` 就可以發現到有兩種不一樣的版本（不是x86的差異），一個叫做 **Window PowerShell** 也就是原生5.1的版本，另一個 **PowerShell** 就是剛安裝好的7這個版本。

![search powershell](search-powershell.png)

## Window Terminal

為什麼有了 PowerShell 這樣的工具，還需要另外一套類似的工具呢？PowerShell 是一個強大的工具，但一次就是一個視窗，分散開來其實有時候要管理的時候不是非常方便，如果同時又開了 cmd 以及 ubnutu wsl 的話，那不就一堆視窗在那邊跑，在這種情況下就蠻適合使用 Window Terminal 這套工具的。

> 好啦，最重要的原因是跟 mac 很像，所以選他就對了！

一樣的微軟提供許多安裝方式，其中官方推薦的是 [Window Store](https://www.microsoft.com/store/productId/9N0DX20HK701) 。

![window terminal select](window-terminal-select.png)

# 套用 Theme

## 安裝

打開剛裝好的 **Window Terminal** 以及 **PowerShell 7**，接下來我們就要開始安裝 Theme 相關的套件，總共要安裝兩個套件，一個是 [oh-my-pos](https://ohmyposh.dev/) 樣式的套件，另一個就是 [posh-git](https://github.com/dahlbyk/posh-git) 是用來顯示 git 的狀態，安裝的指令如下。

```powershell
Install-Module oh-my-posh -Scope CurrentUser
Install-Module posh-git -Scope CurrentUser
```

接著就是要讓 PowerShell 在啟動的時候就能夠套用，需要去修改設定檔，可以透過 `$PROFILE` 這個指令來確認設定檔的位置。如果沒有設定檔，則建立一個並且使用 **記事本** 來開啟。

```powershell
if (!(Test-Path -Path $PROFILE )) { New-Item -Type File -Path $PROFILE -Force }
notepad $PROFILE
```

將以下的內容貼到記事本中，存檔以後，重開一個分頁即可看到套用 `Paradox` 這個樣式。

```powershell
Import-Module posh-git
Import-Module oh-my-posh
Set-PoshPrompt -Theme Paradox
```

## 執行錯誤

> 若你重開分頁以後有套用成功，請跳過這個章節。

若你開啟後看到後面的錯誤，**「檔案無法載入，因為這個系統已停用指令碼執行」**，也不用太過多的擔心，在預設的情況下，PowerShell 是禁止執行自訂的檔案，可以透過 **管理原權限** 來打開 PowerShell 並且執行以下指令

```powershell
Set-ExecutionPolicy RemoteSigned
```

## 套用樣式

oh-my-posh 提供蠻多的樣式可以做選擇，可以透過指令 **Get-PoshThemes** 或是 網站[Themes | Oh my Posh](https://ohmyposh.dev/docs/themes) 來做參考，下圖是執行指令出來的部份結果。

![posh theme](posh-theme.png)

> 如果發現有些內容顯示不出來，通常是字體的問題，可以改為官方使用的字體 [Nerd Fonts](https://www.nerdfonts.com/)

如果預設的樣式還是不滿意的話，也可以自己客製化，做一些調整，讓自己開心工作上也會比較有動力！

```powershell
Export-PoshTheme -FilePath C:\.oh-my-posh.omp.json
```

重新開啟 `$PROFILE` 設定檔，並且將剛剛的第三行 **Set-PoshPrompt** 改成套用客製化的 json 檔案即可。

```powershell
Import-Module posh-git
Import-Module oh-my-posh
Set-PoshPrompt -Theme C:\.oh-my-posh.omp.json
```

最後分享一下我自己的設定檔，是參考了 `paradox` 以及 `jandedobbeleer` 所整理出來的樣式。

![mytheme](mytheme.png)

{% gist 7851a94d92f5a5f35d1c7685afe56d44 .mytheme.omp.json  %}

# 參考

* [使用 oh-my-posh 美化 PowerShell 樣式 (poychang.net)](https://blog.poychang.net/setting-powershell-theme-with-oh-my-posh/)
* [PowerShell 美化：oh my posh | Flymia 凡事用心之事 (ppundsh.github.io)](https://ppundsh.github.io/posts/ad6e/)