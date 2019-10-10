---
title: '[C#] FileSystemWatcher'
date: 2009-04-09 15:47:45
categories:
- Back-end
- C#
tags:
- C#
---
這是一個讓我覺得有趣的元件...
他是用來監控檔案是否有新增刪除修改...
詳細的介紹還是看[msdn](http://msdn.microsoft.com/zh-tw/library/system.io.filesystemwatcher.aspx)吧...
來點小範例...

<!--more-->

``` csharp
void Main()
{
    FileSystemWatcher fsw = new FileSystemWatcher();
    fsw.Path = @"D:\";
    fsw.Filter = "*.txt";
    fsw.Created += new FileSystemEventHandler(fsw_Created);
    fsw.Changed += new FileSystemEventHandler(fsw_Changed);
    fsw.EnableRaisingEvents = true;
 }

void fsw_Changed(object sender, FileSystemEventArgs e)
{
    Console.WriteLine("File: " + e.FullPath + " " + e.ChangeType);
}

void fsw_Created(object sender, FileSystemEventArgs e)
{
    Console.WriteLine("File: " + e.FullPath + " " + e.ChangeType);
}
```