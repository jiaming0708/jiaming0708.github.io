---
title: Telerik RadGrid的GridButtonColumn修改ConfirmText
date: 2011-09-29 09:53
categories:
- Back-end
- ASP.NET
tags:
- ASP.NET
---
Telerik是第三方的元件，加強了前台和後台的一些功能
並且都經過美化，讓開發者可以簡單使用又不失美觀
[telerik官網](http://www.telerik.com/)

我們最常使用的是RadGrid（其實就是GridView的功能）
會用到的Column有GridButtonColumn、GridTemplateColumn、GridBoundColumna等

其中GridButtonColumn這樣設定
``` html
<telerik:gridbuttoncolumn headerstyle-width="20px" confirmtext="Delete this contact?" ConfirmDialogType="RadWindow" ConfirmTitle="Delete" ButtonType="PushButton" CommandName="Delete" Text="Delete" UniqueName="DeleteColumn">
    <itemstyle horizontalalign="Center">
</itemstyle></telerik:gridbuttoncolumn>
```

要修改ConfrimText的內容
需要先了解另一個屬性ButtonType
分為PushButton(Button)、ImageButton(ImageButton)、LinkButton(HyperlinkButton)

這邊就只用PushButton做範例
先找出該物件，物件型別會根據ButtonType有所差異
接著使用原始的給Atrribute作法，來進行修改
``` csharp
protected void rgFile_ItemDataBound(object sender, Telerik.Web.UI.GridItemEventArgs e)
{
    if (e.Item is GridDataItem)
    {
        GridDataItem item = e.Item as GridDataItem;
        Button btnDelete = item["DeleteColumn"].Controls[0] as Button;
 
        if (btnDelete != null)
        {
            btnDelete.Text = _UIFace.GetFaceString("btnDelete");
            //ANS-00002:確定要刪除？
            btnDelete.Attributes["onclick"] = string.Format("return confirm('{0}');", _UIMsg.GetMessageString("ANS-00002"));
        }
    }
}
```