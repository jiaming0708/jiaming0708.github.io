---
title: '[C#] 型別當變數'
date: 2010-03-29 16:58:37
categories:
- Back-end
- C#
tags:
- C#
---
自己再寫MDI的視窗程式時
寫一個檢查子視窗是否開啟的method
想傳form的型別過去當參數

<!--more-->

不知道該如何寫
再網路上問了一下
得到解答如下

就是用typeof轉成Type傳過去
``` csharp
private bool CheckChildrenFormOpened(Type type)
{
    foreach (Form f in this.MdiChildren)
    {
        if (f.GetType() == type)
        {
            f.Activate();
            return true;
        }
    }
 
    return false;
}
 
if (!CheckChildrenFormOpened(typeof(AccountManage)))
{
}
```