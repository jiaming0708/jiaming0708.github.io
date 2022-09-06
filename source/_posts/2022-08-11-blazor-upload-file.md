---
title: '[Blazor] 上傳檔案'
date: 2022-08-11 09:18:50
categories:
- Frontend
- Blazor
tags:
- Blazor
---

本篇會介紹如何在 Blazor 使用 `multipart/form-data` 上傳檔案的功能，並且在 API 要如何接收。

<!-- more -->

## Blazor

在 Blazor 中，有一個專門的元件叫做 `InputFile` 是給檔案上傳使用，其實等於 `<input type="file">`，基本上除了 `OnChnage` 這個是 Blazor 提供的以外，其他原生的屬性都能直接使用；像下面這個範例，就是可以選取多個 xlsx 的檔案。

```html
<InputFile OnChange="@OnInputFileChange" multiple accept=".xlsx" />
```

`InputFileChangeEventArgs` 對於單檔和多檔有不同做法，那不管哪一種都是會拿到 `IBrowserFile` 這個型別的物件。

```c#
private void OnInputFileChange(InputFileChangeEventArgs args)
{
    var singleFile = args.File;
    var multipleFiles = args.GetMultipleFiles();
}
```

接著來準備上傳的物件，要先轉成 stream 然後存到 `MultipartFormDataContent` 這個類別中，加入的時候記得要使用 **files** 作為 name；呼叫 API 時，記得要使用 `PostAsync` 而不是 `PostAsJsonAsync`，content-type 才會是正確的 `multipart/form-data`。

```c#
private void OnInputFileChange(InputFileChangeEventArgs args)
{
    var singleFile = args.File;
    var fileContent = new StreamContent(singleFile.OpenReadStream());
    fileContent.Headers.ContentType = new MediaTypeHeaderValue(file.ContentType);
    
    using var formData = new MultipartFormDataContent();
    // 中間的名稱記得一定要給 files
    formData.Add(fileContent, "files", singleFile.Name);
	// 一定要用 PostAsync 才會是正確的 multipart/form-data
    await HttpClient.PostAsync("api/upload", formData);
}
```

如果要在 `MultipartFormDataContent` 中放入其他非檔案的資料，要先轉換成 `StringContent` 才行。

```c#
using var formData = new MultipartFormDataContent();
formData.Add(new StringContent("jimmy"), "name");
```

## Controller

前端搞定以後，後端的 API 就相對的簡單，只要用 `FromForm` 的屬性來對應就好，檔案的話請用 `List<IFormFIle>` 作為型別也就是對應到前面所寫的 **files**，其他的資料則是各自對應名稱。
使用 FileStream 來寫入到檔案，或是用 MemoryStream 載入到記憶體中。

```c#
[HttpPost]
public async Task<IActionResult> Upload([FromForm] List<IFormFile> files, [FromForm] string name)
{
    var path = $@"{_environment.WebRootPath}\upload\";
    
    foreach(var file in files)
    {
        // 寫入檔案
        await using FileStream fs = new($"{path}{file.FileName}", FileMode.Create);
        await file.CopyToAsync(fs);
        // 放在記憶體
        using var ms = new MemoryStream();
        await file.CopyToAsync(ms);
    }
}
```

API 後續的邏輯就看需要做些什麼，接續著做就好。

## Reference

- [ASP.NET Core Blazor file uploads | Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/blazor/file-uploads?view=aspnetcore-6.0&pivots=server)