---
title: '[windows] 真正關閉Windows 8的UAC'
date: 2014-09-17 20:43:23
categories:
- Windows
tags:
- Windows
---

UAC這東西從Vista就開始存在了
但以前想要把使用者變成真正的Administrator
只要到「使用者帳戶」的管理下把「變更使用者帳戶控制設定」調整為最小即可

<!--more-->

但是在win 8開始，這個並不是真正的Administrator，還是需要手動使用系統管理員執行的方式才能擁有權限
Microsoft希望讓系統的安全性更高，但是這樣對於我來說其實很大的不方便
為了讓自己能夠拿真正的權限，就必須要動手做一些的修改

>1.開啟regedit
2.到「HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System」下找到「Enable LUA」
3.將數值修改為0
4.歡樂使用最高權限