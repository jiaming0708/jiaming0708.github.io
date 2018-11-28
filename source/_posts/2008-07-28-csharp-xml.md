---
title: 讀取xml檔
date: 2008-07-28 11:08:09
categories:
- Back-end
- C#
tags:
- C#
---
以下為xml檔案內容

<!--more-->
``` xml
<?xml version="1.0" encoding="big5"?>
<noval>
	<BookID>001
		<Name>天魔神譚</Name>
		<Writer>手槍</Writer>
		<LastReadDate>2008/07/21</LastReadDate>
		<Chapter>第3部第13集第9章</Chapter>
		<Finish>False</Finish>
	</BookID>
	<BookID>002
		<Name>魔法炒手</Name>
		<Writer>張君寶</Writer>
		<LastReadDate>2008/07/27</LastReadDate>
		<Chapter>第609章</Chapter>
		<Finish>True</Finish>
	</BookID>
	<BookID>003
		<Name>異俠</Name>
		<Writer>自在</Writer>
		<LastReadDate>2008/07/27</LastReadDate>
		<Chapter>第25集第8章</Chapter>
		<Finish>False</Finish>
	</BookID>
</noval>
```
以下為片段程式碼
``` csharp
using System.Xml;

public XmlDocument myXml = new XmlDocument();
myXml.Load(strFileName);//讀取xml檔
numBook = myXml.ChildNodes.Item(1).ChildNodes.Count;//總書本數
for (int i = 0; i < numBook; i++)
{
   cmbBook.Items.Add(myXml.ChildNodes.Item(1).ChildNodes.Item(i).ChildNodes.Item(1).InnerText);
}
```