---
title: WinForm的DataGridView類似Web的GridView_DataRowBound事件
date: 2012-05-17 20:50:37
categories:
- Back-end
- C#
tags:
- C#
---
最近在寫自己的小程式
不過是使用WinForm開發，跟工作時使用的Web不一樣
蠻多事件都不太一樣

其中，在Web上最好用的事件，GridView_DataRowBound
在WinForm的DataGridView竟然沒有><
後來東找找西找找，找到了一個類似的
這個方法是一個儲存格一個儲存格的處理，但Web是一個Row一起處理，還是Web好用阿~~~~

讓我們來看看如何使用吧
``` csharp
private void dgvData_CellFormatting(object sender, DataGridViewCellFormattingEventArgs e)
{
    try
    {
        if (!dgvData.Columns[e.ColumnIndex].Name.Equals("Status", StringComparison.OrdinalIgnoreCase))
            return;
 
        if (e.Value == "Y")
            e.Value = "是";
        else
            e.Value = "否";
    }
    catch (Exception ex)
    {
        WriteMessage(ex);
    }
}
```