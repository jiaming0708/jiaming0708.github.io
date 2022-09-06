---
title: '[windows] 7/8變更使用者目錄'
date: 2013-09-13 23:42:33
categories:
- Windows
tags:
- Windows
---
好久沒有寫點東西，來分享一個實用的東西

<!--more-->

win 7/8的User資料夾會存放系統中很多暫存的東西
當系統越用越久，就會越來越肥

如果系統槽很不巧的只有給一點點的空間大小
那慘了，會看到系統跟你說空間不足
最後可能就使用系統都有問題

接下來告訴大家一個好方法
但是 but 旦夕，這招只能用在剛安裝系統時

1. 在安裝過程中，要求輸入用戶名及密碼時（或是輸入電腦名稱時），先不輸入按「Shift+F10」呼出DOS視窗（注意的是更換的槽必須為NTFS）
2. robocopy "C:\Users" "X:\Users" /E /COPYALL /XJ
3. rmdir "C:\Users" /S /Q
4. mklink /J "C:\Users" "X:\Users"
5. 關閉DOS窗口，繼續安裝至完成，這樣安裝的系統，所有「User」的內容都將設置在X槽。