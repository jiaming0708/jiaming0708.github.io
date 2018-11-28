---
title: 參數傳遞觀念
date: 2009-02-12 22:44:41
categories:
- Back-end
- C#
tags:
- C#
---
這是從同事那邊得知的知識...
原因是同事寫了一段程式讓我十分疑惑...

<!--more-->

``` csharp
class Form1
{
 public void Main()
 {
  DataSet oDataSet=new DataSet();
  Function.LoadDbList(DbName, oDataSet);
  for(int i=0;i<oDataSet.Table["ALARMLIST"].count;i++)
  {
   //...
  }
 }
}

class Function
{
 public static void LoadDbList(string DbName, DataSet oDataSet)
 {
  //...
  dataAdapter.Fill(oDataSet);
 }
}
```

在執行完Function.LoadDbList後，竟然就可以直接使用oDataSet這個物件
但是程式中oDataSet這個物件並沒有回傳，也沒有ref或是out的用法
但是就有辦法在接下的程式中使用

後來問同事後得知，原來除了基本的型態(string,int...等)
其他的型態都是用reference的方式在傳遞參數

這樣就可以不必用return的方式
聽同事說可以不用在new一個物件，似乎可以避免浪費資源