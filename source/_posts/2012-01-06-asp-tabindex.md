---
title: 使用Tab不停留在物件上
date: 2012-01-06 14:13:07
categories:
- Back-end
- ASP.NET
tags:
- ASP.NET
---
這個功能很單純，只是希望按Tab鍵的時候
能夠跳過某些物件，按照希望的順序來切換

<!--more-->

不停留TabIndex = -1，其他順序TabIndex=1,2...

``` html
<!-- Tab停留 -->
<asp:textbox id="ttbUser" runat="server" tabindex="1">
</asp:textbox>
<!-- Tab不停留 -->
<asp:textbox id="ttbUserName" runat="server" tabindex="-1">
</asp:textbox>
```