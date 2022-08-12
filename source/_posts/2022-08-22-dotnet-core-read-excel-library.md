---
title: '[DotnetCore] 讀取 excel 套件整理'
date: 2022-08-22 11:07:00
categories:
- Backend
- DotnetCore
tags:
- DotnetCore
---

dotnet core 的專案現在有讀取 excel 的需求了，測試幾套後整理做法，給有需要的人。總共看了四套，其中一套在商用上是需要付費的，使用上要注意一下。

<!-- more -->

Excel 的資料格式如下，並且定義類別。

![excel-sample](excel-sample.png)

```c#
class Data
{
	public string WIP {get;set;}
	public string WIP_STATUS {get;set;}
	public Detetime DEADLINE {get;set;}
}
```

## OpenXML

[OfficeDev/Open-XML-SDK: Open XML SDK by Microsoft (github.com)](https://github.com/OfficeDev/Open-XML-SDK)，由微軟自己所提供，也算是蠻成熟的套件，缺點是寫起來蠻囉嗦的；在 nuget 中搜尋 `DocumentFormat.OpenXml` 並且安裝到專案中。

如果是字串的話，資料會放在 [ShareString Table](https://docs.microsoft.com/en-us/office/open-xml/working-with-the-shared-string-table?redirectedfrom=MSDN)，一開始取得的資料就會是 index。日期的話可以用 `DateTime.FromOADate` 做處理。

```c#
var path = @"D:\Import_WIP_Example.xlsx";
using var doc = SpreadsheetDocument.Open(path, false);
var workbookPart = doc.WorkbookPart;
var sheet = workbookPart.Workbook.Sheets.GetFirstChild<Sheet>();
var worksheet = ((WorksheetPart)workbookPart.GetPartById(sheet.Id)).Worksheet;
var sheetData = worksheet.GetFirstChild<SheetData>();
var sharedTable = workbookPart.SharedStringTablePart.SharedStringTable;
// 使用物件來承接資料
var list = new List<Data>();

// 跳過欄位名稱的 row1
foreach (var row in sheetData.ChildElements.Select(x => x as Row).Skip(1))
{
	var data = new Data();

	for (var i = 0; i < row.ChildElements.Count; i++)
	{
		var cell = row.ChildElements[i] as Cell;
         // 取得欄位資料
		var innerText = cell.InnerText;
         // 只有字串的資料會在 SharedString Table，需要另外取資料
		if (cell.DataType != null && cell.DataType == CellValues.SharedString)
		{
			SharedStringItem item = sharedTable.Elements<SharedStringItem>().ElementAt(Int32.Parse(innerText));
			innerText = item.InnerText;
		}
         // 預期 C 欄位是日期，可以用 FromOADate 來轉成 DateTime
		if(cell.CellReference.Value.StartsWith("C") && Int32.TryParse(innerText, out var oaDate)){
			data.DEADLINE = DateTime.FromOADate(Convert.ToInt32(oaDate));
			continue;
		}
         // 寫入到物件
		switch (i)
		{
			case 0:
				data.WIP = innerText;
			break;
			case 1:
				data.WIP_STATUS = innerText;
			break;
		}
    }
    // 去除空白資料
	if(data.WIP == "" && data.WIP_STATUS == "") continue;
	list.Add(data);
}
// 使用 LINQPad 的話可以解除註解來看資料
// list.Dump();
```

## NPOI

[nissl-lab/npoi: a .NET library that can read/write Office formats without Microsoft Office installed. No COM+, no interop. (github.com)](https://github.com/nissl-lab/npoi)，在 nuget 搜尋 `NPOI` 並安裝。

相對於 OpenXML 的優點是

- 可以直接移除空白列
- 更好的日期處理

```c#
var path = @"D:\Import_WIP_Example.xlsx";
await using FileStream fs = new(path, FileMode.Open);
var xssWorkbook = new XSSFWorkbook(fs);
var sheet = xssWorkbook.GetSheetAt(0);
IRow headerRow = sheet.GetRow(0);
int cellCount = headerRow.LastCellNum;
// 使用物件來承接資料
var list = new List<Data>();

for (var i = (sheet.FirstRowNum + 2); i <= sheet.LastRowNum; i++)
{
	var row = sheet.GetRow(i);
	if (row == null) continue;
	// 移除空白列
	if (row.Cells.All(d => d.CellType == CellType.Blank)) continue;

	var data = new Data();
	list.Add(data);

	for (var j = row.FirstCellNum; j < cellCount; j++)
	{
		var cell = row.GetCell(j);
		if (cell == null) continue;
         // 日期處理
		if (cell.CellType == CellType.Numeric && DateUtil.IsCellDateFormatted(cell))
		{
			data.DEADLINE = cell.DateCellValue;
			continue;
		}
		
		switch (j)
		{
			case 0:
				data.WIP = cell.ToString();
			break;
			case 1:
				data.WIP_STATUS = cell.ToString();
			break;
		}
	}
}
// 使用 LINQPad 的話可以解除註解來看資料
// list.Dump();
```

## EPPlus

[EPPlusSoftware/EPPlus: EPPlus 5-Excel spreadsheets for .NET (github.com)](https://github.com/EPPlusSoftware/EPPlus)，**商用需要付費**；在 nuget 中搜尋 `EPPlus` 並且安裝。

- 可以快速移除空白列
- 資料型態可以判斷

```c#
var path = @"D:\Import_WIP_Example.xlsx";
ExcelPackage.LicenseContext = LicenseContext.Commercial;
using var package = new ExcelPackage(new FileInfo(path));

var sheet = package.Workbook.Worksheets[0];
var startRowNumber = sheet.Dimension.Start.Row;
var endRowNumber = sheet.Dimension.End.Row;
var startColumn = sheet.Dimension.Start.Column;
var endColumn = sheet.Dimension.End.Column;

var list = new List<Data>();

for (var currentRow = startRowNumber + 1; currentRow <= endRowNumber; currentRow++)
{
    // 移除空白列
	ExcelRange range = sheet.Cells[currentRow, startColumn, currentRow, endColumn];
	if (range.All(c => !string.IsNullOrEmpty(c.Text)) == false) continue;

	var data = new Data();
	list.Add(data);

	for (int j = startColumn; j <= endColumn; j++)
	{
         // 使用 Value 可以拿到日期型態
		var cellValue = sheet.Cells[currentRow, j].Value;
         // 欄位是從 1 開始
		switch (j)
		{
			case 1:
				data.WIP = cellValue.ToString();
			break;
			case 2:
				data.WIP_STATUS	= cellValue.ToString();
			break;
			case 3:
				data.DEADLINE = (DateTime)cellValue;
			break;
		}
	}
}
// 使用 LINQPad 的話可以解除註解來看資料
// list.Dump();
```

## MiniExcel

[MiniExcel/MiniExcel: Fast, Low-Memory, Easy Excel .NET helper to import/export/template spreadsheet (github.com)](https://github.com/MiniExcel/MiniExcel)，在 nuget 搜尋 `MiniExcel` 並安裝。

算是比較新的套件，可以直接轉成物件，寫起來簡潔明瞭。

```c#
var path = @"D:\Import_WIP_Example.xlsx";
var list = await MiniExcel.QueryAsync<Data>(path, startCell: "A1");
list = list.Where(p => p.WIP is not null && p.WIP_STATUS is not null);
// 使用 LINQPad 的話可以解除註解來看資料
// list.Dump();
```

## 結論

以上四套，大家就看看自己的需求來使用囉，前面三套都蠻多資料可以查詢，只有最後一套 MiniExcel 就比較少，個人最後是選用 MiniExcel 這套，主要是可以搭配 EF 更快速的寫入資料庫。

## Reference

- [使用Open XML SDK讀取Excel中的文字-黑暗執行緒 (darkthread.net)](https://blog.darkthread.net/blog/open-xml-sdk-read-excel-string/)
- [[VB\] NPOI - 讀寫Excel檔案 - HackMD](https://hackmd.io/@carlochuang/npoi)
- [[C#\] EPPlus 讀寫(read/write) Excel檔案 懶人包範例程式碼 | 高級打字員的技術雲 - 點部落 (dotblogs.com.tw)](https://dotblogs.com.tw/shadow/2012/07/13/73385)
