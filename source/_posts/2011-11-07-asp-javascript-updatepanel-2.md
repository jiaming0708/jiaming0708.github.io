---
title: '[ASP.NET] 在UpdatePanel裡執行JavaScript II'
date: 2011-11-07 19:08:05
categories:
- Backend
- ASP.NET
tags:
- ASP.NET
---
上次發表這篇 {% post_link asp-javascript-updatepanel 在UpdatePanel裡執行JavaScript %}
只是提到如何在UpdatePanel裡面，在後台中去執行前台的Script

<!--more-->

那如果是要每次PostBack後都需要執行呢
那就必須改用這個方式

這段必須放在Body裡面，這要特別注意

``` html
<script>
function hello()
{
   alert('hello!');
}
</script>

Sys.WebForms.PageRequestManager.getInstance().add_endRequest(hello);
```