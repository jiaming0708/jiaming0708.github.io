---
title: '[C#] 按tab+alt切換視窗時隱藏圖示'
date: 2009-01-06 00:07:48
updated: 2009-01-06 00:07:48
categories:
- Backend
- C#
tags:
- C#
---
用到這個的情況是在...
一隻AP是不顯示在工作列的...
但是卻又常駐在桌面...

<!--more-->

要讓圖示消失在切換的工作列中...
其實有一個最簡單的設定...
只需要兩行...

``` csharp
this.FormBorderStyle = System.Windows.Forms.FormBorderStyle.FixedToolWindow;
this.ShowInTaskbar = false;
```

但是當我設定改為...

``` csharp
this.FormBorderStyle = System.Windows.Forms.FormBorderStyle.None;
```

圖示一樣會出現...
這樣就無法達到我的需求...

同事有一隻AP擁有這樣的功能...
不過他的需求不同...
他是要在按下顯示桌面時...
程式不會跟著縮小...

廢話就不多說了...
還是來看看怎樣達到這樣的目的...

``` csharp
[DllImport("USER32.DLL")]
public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
[DllImport("USER32.DLL")]
public static extern IntPtr FindWindowEx(IntPtr parentHandle, IntPtr childAfter, string className, IntPtr windowTitle);
[DllImport("USER32.DLL")]
public static extern IntPtr SetParent(IntPtr hWndChild, IntPtr hWndNewParent);

IntPtr hdesk = FindWindow("Progman", "Program Manager");
hdesk = FindWindowEx(hdesk, IntPtr.Zero, "SHELLDLL_DefView", IntPtr.Zero);
hdesk = FindWindowEx(hdesk, IntPtr.Zero, "SysListView32", IntPtr.Zero);
SetParent(this.Handle, hdesk);
```

不過用了這個以後...
原本所寫的top功能就消失了...
這個功能還要在找尋...