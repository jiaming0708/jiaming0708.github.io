---
title: '[ASP.NET] 在UpdatePanel裡使用FileUpload控制項'
date: 2012-09-20 21:00:12
updated: 2012-09-20 21:00:12
categories:
- Backend
- ASP.NET
tags:
- ASP.NET
---
一般使用FileUpload控制項時，只要PostBack回後台，有上傳的檔案一定取的到
但是某天我在外面包了一個UpdatePanel

<!--more-->

慘案發生了
``` html
<asp:updatepanel id="UpdatePanel1" runat="server">
<contenttemplate>
    <asp:fileupload id="FileUpload1" runat="server">
    <asp:button id="Button1" runat="server" text="Button">
</asp:button></asp:fileupload></contenttemplate>
</asp:updatepanel>
```

PostBack回到後台接不到上傳的檔案orz
還好有同事愈過這樣的情況

同事說只要加上Trigger就可以了，馬上測試

``` html
<asp:updatepanel id="UpdatePanel1" runat="server" updatemode="Conditional">
<contenttemplate>
    <asp:fileupload id="FileUpload1" runat="server">
    <asp:button id="Button1" runat="server" text="Button">
</asp:button></asp:fileupload></contenttemplate>
<triggers>
<asp:postbacktrigger controlid="Button1">
</asp:postbacktrigger></triggers>
</asp:updatepanel>
```


捷克～真的可以了
原來是FileUpload一定要觸發PostBack，後台才能取得上傳的檔案
[MSDN中有說到，FileUpload和UpdatePanel不相容](http://msdn.microsoft.com/zh-tw/library/bb386454.aspx)
學藝不精阿～還是要多多看MSDN