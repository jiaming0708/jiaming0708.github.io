---
title: '[ASP.NET] 用js開啟AJAX Accordion'
date: 2010-08-10 10:31:42
categories:
- Backend
- ASP.NET
tags:
- ASP.NET
---
專案有一個需求
要點了某個物件以後去顯示相關內容(那內容放在Accordion的其中一個panel)
我採用了js的ajax的方式去取得
主管又希望能夠在點了那個物件後
顯示相關內容那塊能顯示出來

<!--more-->

由於我都是在前台作業
沒有postback到後台
因此這個需求就出現了

這段必須加在form裡面，不然$get會抓不到那個物件
``` js
function Openpane(paneIndex) {
    //Accordion is the ID of AJAX Accordion control
    var behavior = $get("<%=EmsAccordion.ClientID%>").AccordionBehavior;
    behavior.set_SelectedIndex(paneIndex);
}
```