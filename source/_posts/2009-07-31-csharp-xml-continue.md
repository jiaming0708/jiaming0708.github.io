---
title: xml讀取寫入(續)
date: 2009-07-31 00:10:11
categories:
- Back-end
- C#
tags:
- C#
---
前一篇來看的人很多...
但應該會覺得介紹的太少了...
這次來完整一點吧...
算是自己的一個整理...

xml內容...(這是我記帳程式的格式...)

``` xml
<?xml version="1.0" encoding="Big5"?>
<MoneyManage>
  <UserClassification>
    <Category Name="餐飲" ItemCount="8" Key="1">
      <Item Name="早餐" Category="1" Type="支出" Comment="" Key="101" />
      <Item Name="中餐" Category="1" Type="支出" Comment="" Key="102" />
      <Item Name="晚餐" Category="1" Type="支出" Comment="" Key="103" />
      <Item Name="點心宵夜" Category="1" Type="支出" Comment="" Key="104" />
      <Item Name="飲料" Category="1" Type="支出" Comment="" Key="105" />
      <Item Name="其他" Category="1" Type="支出" Comment="" Key="119" />
      <Item Name="水果" Category="1" Type="支出" Comment="" Key="151" />
      <Item Name="麵包" Category="1" Type="支出" Comment="" Key="152" />
    </Category>
    <Category Name="交通" ItemCount="6" Key="2">
      <Item Name="加油" Category="2" Type="支出" Comment="" Key="206" />
      <Item Name="坐車" Category="2" Type="支出" Comment="" Key="207" />
      <Item Name="換機油" Category="2" Type="支出" Comment="" Key="208" />
      <Item Name="停車費" Category="2" Type="支出" Comment="" Key="233" />
      <Item Name="過路費" Category="2" Type="支出" Comment="" Key="235" />
      <Item Name="修理" Category="2" Type="支出" Comment="" Key="242" />
    </Category>
  </UserClassification>
</MoneyManage>
```

開始運用...
取得所有Category...

## 第一種方法
``` csharp
XmlNode root = xmlDoc.SelectSingleNode("/MoneyManage/UserClassification");
foreach (XmlElement elm in root.ChildNodes)
{
     Label1.Text += "Category Name:" + elm.GetAttribute("Name");
}
```


## 第二種方法
XmlNodeList root = xmlDoc.SelectNodes("/MoneyManage/UserClassification");
foreach (XmlElement elm in root)
{
     Label1.Text += "Category Name:" + elm.GetAttribute("Name");
}
```csharp

接下來是要取得名稱為"停車費"的Item...

``` csharp
XmlElement node = (XmlElement)xmlDoc.SelectSingleNode("/MoneyManage/UserClassification/Category/Item[@Name=\"停車費\"]");
Label1.Text += "Category Name:" + node.GetAttribute("Name");
```

如果要多個條件...
類似SQL...
[@Key = ? and @Name = ?]

讀取的部份算是差不多...
接下來就是寫入的部份啦...

``` csharp
XmlNode root = xmlDoc.SelectSingleNode("/MoneyManage/UserClassification");
XmlNode node = xmlDoc.CreateNode(XmlNodeType.Element, "Category", null);

XmlAttribute attribute = Variable.XmlDoc.CreateAttribute("Name");
attribute.Value = "住宿";

node.Attributes.Append(attribute);
root.AppendChild(node);

xmlDoc.Save("jiaming.xml");
```