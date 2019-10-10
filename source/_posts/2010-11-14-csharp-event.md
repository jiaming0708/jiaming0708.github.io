---
title: '[C#] 自己寫Event'
date: 2010-11-14 09:36:39
categories:
- Back-end
- C#
tags:
- C#
---
好久沒有來寫點東西了...
換了工作以後被鳥客戶纏身...
每天就是忙盲茫...

<!--more-->

廢話不多說...
直接進入正題吧...

這次的需求是這樣的...
想要去寫入log資訊...
但又不希望每個class都要去加上寫入log的function...
因此採用Event的方式...

``` csharp
class Program
{
    static void Main(string[] args)
    {
        EventListen.OnLoggin += new EventListen.LogHandler(EventListen_OnLoggin);
        EventListen el = new EventListen();
        Console.ReadLine();
    }
 
    static void EventListen_OnLoggin(string message)
    {
        Console.WriteLine(message);
        Console.ReadLine();
    }
}

public class EventListen
{
    public delegate void LogHandler(string message);
    public static event LogHandler OnLoggin;
 
    public EventListen()
    {
        //避免event沒有被listen
        if (OnLoggin != null)
        {
            OnLoggin("test");
        }
    }
}
```