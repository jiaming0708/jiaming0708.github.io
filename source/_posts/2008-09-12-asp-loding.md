---
title: '[ASP.NET] 跳出類似loading的頁面'
date: 2008-09-12 22:46:36
categories:
- Backend
- ASP.NET
tags:
- ASP.NET
---
這是在改公司網頁時看到的一個小功能... 
感覺好像可以拿出來應用... 
就先把他紀錄起來了...

<!--more-->

``` html
<INPUT id="ReturnValueObj" style="Z-INDEX: 119; LEFT: 8px; WIDTH: 16px; POSITION: absolute; TOP: 264px; HEIGHT: 16px"
    type="hidden" size="1" name="Hidden1" runat="server"> 
```

``` csharp
//全域變數
protected System.Web.UI.HtmlControls.HtmlInputHidden ReturnValueObj;
 
//呼叫method
ShowDialogBox (ReturnValueObj ,"WebForm1.aspx",200,50,0,0,true);
 
//method函式
private void ShowDialogBox(System.Web.UI.HtmlControls.HtmlInputHidden ReturnValueObj,string sURL ,int width ,int height, int x,int y, Boolean isCenter )
  {
   string sFeature="";
   string sJs="";
   //iCenter = false;
   sFeature += "dialogHeight:" + height + "px;";
   sFeature += "dialogWidth:" + width + "px;";
   if (isCenter == false) 
   {
    sFeature += "dialogLeft:" + x + "px;";
    sFeature += "dialogTop:" + y + "px;";
   }
   sJs = "<script>";
   sJs += "Form1." + ReturnValueObj.ClientID + ".value =window.showModalDialog('WebForm2.aspx?url=" + sURL + "','','" + sFeature + "');";
   sJs += "</script>";
   this.RegisterStartupScript("ShowDialogBox",sJs);
   //把hidden textbox 的值接回window.alert 上
   string script ="<script>";
   script +="if(Form1." + ReturnValueObj.ClientID + ".value != \"\") { window.alert(Form1." + ReturnValueObj.ClientID + ".value); } ";
   script +="</script>";
   this.RegisterStartupScript("ShowAlert",script);
  }
```