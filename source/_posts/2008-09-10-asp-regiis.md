---
title: 無法執行要求，因為無法建立 App-Domain
date: 2008-09-10 22:52:14
categories:
- Back-end
- ASP.NET
tags:
- ASP.NET
---
公司的電腦要跑asp.net...
但是不能執行，出現這個錯誤訊息...
不過還是沒解決問題...

<!--more-->

因為是copy過來的不能run...
但開新的可以run...
最後同事說直接開一個新專案...
然後再把檔案copy過去就可以了...

無法執行要求，因為無法建立 App-Domain。錯誤: 0x80131902 

這是安裝IIS6要跑ASP.NET 2.0時所碰到的問題 
解法如下 
1. With a command window, get to the latest version of .net under 
2. C:\Windows\Microsoft.Net\Framework\ 
3. Now run the following command: "net stop w3svc" to stop web services. 
4. Then use "aspnet_regiis.exe -ua" to uninstall all instances of ASP.NET from IIS. 
5. Follow with "aspnet_regiis.exe -i" to install ASP.NET into IIS. 
6. Now restart web services with "net start w3svc". 