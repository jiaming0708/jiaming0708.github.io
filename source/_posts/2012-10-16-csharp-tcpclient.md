---
title: TCPClient如何判斷已斷線
date: 2012-10-16 21:04:42
categories:
- Back-end
- C#
tags:
- C#
---
TCPClient有一個屬性Connected可以用來判斷是否還有連線
但是每次斷線後這個屬性還是沒有改變

<!--more-->

MSDN中有說明

> 如果最近一次的作業是將 Client 通訊端連接至遠端資源，則為 true，否則為 false。
Connected 屬性會取得上次 I/O 作業的 Client 通訊端連接狀態。
當它傳回 false 時，即表示 Client 通訊端不是從未連接過，就是不再連接了。
因為 Connected 屬性只反映最近一次作業的連接狀態，所以您應嘗試傳送或接收訊息，以判斷目前的狀態。
訊息傳送失敗之後，這個屬性就不再傳回 true。請注意，這種行為是設計上的預期行為。
您可能無法很穩定地測試連接的狀態，原因是有可能在測試和收發 (訊息) 之間就失去該連接。
您的程式碼應假設該通訊端是連接的，然後再小心處理傳輸失敗的情況。

Connected是記憶最後的連線狀態，通常都是上次連線正常，然後就突然死掉
這樣是要怎麼玩阿orz

可以使用socket的peek功能來試讀資料
用這樣可以得知連線是否還在
``` csharp
try
{
    client.Client.Poll(0, SelectMode.SelectRead);
    byte[] testRecByte = new byte[1];
 
    while (client.Connected)
    {
        if (client.Available == 0)
        {
            //使用Peek，測試client是否還有連線
            if (client.Client.Receive(testRecByte, SocketFlags.Peek) == 0)
                break;
 
            Thread.Sleep(20);
            continue;
        }
    }
}
catch (SocketException ex)
{
}
```

## 參考資料
黑暗執行續 [偵測TcpClient連線狀態](http://blog.darkthread.net/post-2011-08-11-detected-tcpclient-connection-status.aspx)