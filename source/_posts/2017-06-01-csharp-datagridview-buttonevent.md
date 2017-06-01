---
title: Win Form的DataGridView的button事件
date: 2017-06-01 20:44:00
categories:
- Back-end
- C#
tags:
- C#
---
## 說明
新公司剛好需要先寫一個winform只好先硬幹了
習慣了web form的很多東西，其實win form並沒有像web那樣的方便，或者說他其實更加的彈性
很多行為模式都不一樣因此要熟悉一下

其中有個行為其實我們很常用到，那就是在Grid中放了一些元件，然後要觸發事件，像是button的click
在DataGridView中，其實沒有那些事件可以直接使用，因此我們要繞點路
使用CellContentClick這個事件

來看看[MSDN](https://msdn.microsoft.com/zh-tw/library/system.windows.forms.datagridview.cellcontentclick(v=vs.110).aspx)的說明吧
>按一下儲存格內的內容時發生。

非常簡短，也非常好懂，就是當儲存格被按到時會觸發
也許你會問到，這樣不是目標被按到的時候不就也會進去事件？
是的，這也很無奈╮(－_－)╭

## 範例
讓我們來看一下範例吧
```csharp
private void DataGridView1_CellContentClick(object sender, DataGridViewCellEventArgs e)
{
    var colname = DataGridViewLicenseList.Columns[e.ColumnIndex].Name
    if (colname.ToLower() == "edit")
    {
        //do something
    }
}
```

## 相似文章
關於DataGridView的其他操作，之前也有寫過其他篇
{% post_link 2012-05-17-csharp-datagridview-datarowbound WinForm的DataGridView類似Web的GridView_DataRowBound事件 %}