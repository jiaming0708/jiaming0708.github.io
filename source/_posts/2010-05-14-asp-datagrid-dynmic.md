---
title: '[ASP.NET] DataGrid動態產生Template(更新)'
date: 2010-05-14 21:40:30
updated: 2010-05-14 21:40:30
categories:
- Backend
- ASP.NET
tags:
- ASP.NET
---
這是舊的文章...
但這個功能之前其實算是失敗的...
在update時後想要再後台去抓取那些動態生成的欄位值...
但發現都抓不到...
最近都在用jQuery...
就改在前台抓完值後存在某個地方...
再由後台去處理...

<!--more-->

## 舊的內容
為了不想要每次資料庫增加一個欄位...
就要大費工夫的去改一堆程式...
因此想說用動態的去產生要顯示的欄位...

這是ASP1.1的作法...

``` csharp
void Page_Load()
{
 TemplateColumn col = new TemplateColumn();
 col.ItemTemplate = new DataItemTemplate(ListItemType.Item, colu, attr);
 col.EditItemTemplate = new DataItemTemplate(ListItemType.EditItem, colu, attr);
 datagrid.Columns.Add(col);
}
 
 public class DataItemTemplate:ITemplate
 {
  private ListItemType mType;
  private string mColumnName;
  private string mIDName;
 
  public DataItemTemplate(ListItemType type, string colname, string idname)
  {
   mType = type;
   mColumnName = colname;
   mIDName = idname;
  }
 
  public void InstantiateIn(Control container)
  {
   // TODO:  加入 DataItemTemplate.InstantiateIn 實作
   switch (this.mType)
   {
    case ListItemType.Item:
     //可自行改成別的物件
     Label lb = new Label();
     lb.DataBinding += new EventHandler(DataBinding);
     container.Controls.Add(lb);
     break;
    case ListItemType.EditItem:
     //可自行改成別的物件
     DropDownList ddl = new DropDownList();
     ddl.Items.Add("TRUE");
     ddl.Items.Add("FALSE");
     ddl.ID = "ddl" + this.mIDName;
     container.Controls.Add(ddl);
     break;
   }
  }
 
  void DataBinding(object sender, EventArgs e)
  {
   //要對應Item的DataBinding
   Label lb = (Label)sender;
   DataGridItem container = (DataGridItem)lb.NamingContainer;
   object dataValue = DataBinder.Eval(container.DataItem, mColumnName);

   if(dataValue != DBNull.Value)
   {
    lb.Text = dataValue.ToString();
   }
  }
```

## 新的內容
ASP2.0的作法請產考這篇文章...
[http://blog.blueshop.com.tw/hent/archive/2008/02/24/54382.aspx](http://blog.blueshop.com.tw/hent/archive/2008/02/24/54382.aspx)

前台抓取動態欄位值
``` js
//在DataGrid裡面找到叫做update的元件
$('#dg_ControlStatus').find('a[id$=btnUpdate]').click(function(){
    //input Reason
    var tempReason = prompt('Modify Reason?','');
    if(tempReason != null && tempReason != '')
        $('#txtReason').val(tempReason);
    else
        return false;

    var data = '';
 
    //找出該行中的所有儲存格
    $(this).parents('tr').find('td').each(function(){
        //只要CoboBox的值
        if($(this).find('select').length > 0)
            data += $(this).find('select').val() + ',';
    });
 
    //儲存起來給後台處理
    $('#txtSaveData').val(data);
});
```