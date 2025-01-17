---
title: '[C#] ComboBox和ListBox如何將顯示和值分開'
date: 2009-10-08 23:12:56
updated: 2009-10-08 23:12:56
categories:
- Backend
- C#
tags:
- C#
---
今天在寫AP時使用了ComboBox和ListBox
想要在顯示和值是不同的
發現了另一種寫法

<!--more-->

一種寫法是將資料放到DataTable

``` csharp
DataTable dt = QueryTable(sql);

ComboBox cmb = new ComboBox();
cmb.DataSource = dt;
cmb.DisplayMember = "Name";
cmb.ValueMember = "ID";
```

另一種寫法，自己寫一個class去存放

``` csharp
public class ObjectBindItem
{
    private string mDisplay;
    private string mValue;

    public ObjectBindItem(string dis, string val)
    {
        this.mDisplay = dis;
        this.mValue = val;
    }

    public string Display
    {
        get { return this.mDisplay; }
    }

    public string Value
    {
        get { return this.mValue; }
    }

    public override string ToString()
    {
        return this.mDisplay + "-" + this.mValue;
    }
}

ArrayList ary = new ArrayList();
ary.Add(new ObjectBindItem("john", "A123");

ComboBox cmb = new ComboBox();
cmb.DataSource = ary;
cmb.DisplayMember = "Display";
cmb.ValueMember = "Value";
```