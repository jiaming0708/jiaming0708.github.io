---
title: '透過visual studio建立安裝檔案'
date: 2021-11-06 15:37:49
updated: 2021-11-06 15:37:49
categories:
- Backend
tags:
- Backend
- installer
- VS2019
---

前一篇  {% post_link browser-open-application %} 中講到開啟應用程式，接著要來講怎麼樣封裝應用程式在安裝檔內。

<!-- more -->

> 本篇使用的VS版本為 2019 community 英文版 （會選用英文的原因是為了有錯誤訊息時比較好找）

# 封裝應用程式

## 新增Installer專案

在VS 2019中預設是沒有這種的專案類型，必須要另外安裝，可以點擊 [這個網址](https://marketplace.visualstudio.com/items?itemName=VisualStudioClient.MicrosoftVisualStudio2017InstallerProjects) 下載或是在 `Extension Manager` 搜尋 `Installer Project` 

> 記得要先關掉 vs 才會開始安裝

![install-extension](install-extension.png)

安裝重開方案後，要來新增專案，在專案範本中找到 `Setup Project` 並且建立

![add-installer-project](add-installer-project.png)

> 建立好後只要在專案點選編譯，就會有 install 的選項可以安裝測試了，接下來每一個步驟都會建議編譯後安裝確認效果

## 專案設定

建立好專案以後，可以看到一個被開啟的畫面，裡面包含下面幾個項目

* Application Folder
* User's Desktop
* User's Programs

首先要將原先的專案輸出結果放進來，點選 `Application Folder` 右鍵新增 `Project Output` 

![add-project-output](add-project-output.png)

接著來改一些基本的資料，在**方案總管**點選專案再開啟**屬性視窗**

![project-properties](project-properties.png)

> 在設定 Setup Project 的過程中，建議可以把**屬性視窗** pin 起來，還蠻常會用到的

## 加上註冊碼

在專案點選右鍵找到 `Registry` ，開啟註冊碼的設定畫面

![view-menu](view-menu.png)

![registry](registry.png)

根據上一篇的作法，要在 `HKEY_CLASSES_ROOT` 底下建立，這邊我使用專案屬性中的 Manufacturer 作為 custom protocol。

* 專案屬性可以使用 [] 來取得
* 預設的值，只要將名稱留空即可
* command 的執行檔名需要寫死

![registry-url-protocol](registry-url-protocol.png)

> 記得將刪除時移除註冊碼的部份啟用，這樣才不會有垃圾註冊碼在裡面

## 增加安裝步驟

預設的安裝步驟其實很簡易，就是選安裝範圍、路徑，最後就是安裝。

在這邊我準備要新增一個步驟來輸入值，並且將資料寫入到 config 中。
從專案點選右鍵找到 `User Interface` 。

![user-interface](user-interface.png)

在 Start 的區塊點右鍵 **Add Dialog** ，從清單中選擇一個項目來做為預計要新增的步驟，在這邊我選擇的是 `Textboxes(A)` 。

![user-interface-add-dialog](user-interface-add-dialog.png)

就可以看到 Start 的區塊增加了一個 `Textboxes(A)` 在最後一個項目，可以用滑鼠拖拉改變順序到第三個步驟。

![user-interface-textbox](user-interface-textbox.png)

接著打開 **屬性視窗** 來修改基本設定，可以看到可調整的程度很低，只有四個輸入框及畫面的標題這幾個項目而已，連位置什麼的都不能改變。（這邊就先不改任何設定）

![user-interface-textbox-properties](user-interface-textbox-properties.png)

## 使用 Custom action 修改 exe.config

前面所講到的都是透過介面的方式來操作，在這個專案中提供 Custom action 的功能，可以透過寫程式的方法來做更多的事情。

### 設定 Custom action

同樣的在專案右鍵中找到 `Custom Actions` 並且開啟，並且在 `Install` 的地方點選右鍵 `Add Custom Action`，選擇前面在 `Application Folder` 中所建立的 Output 即可。

![custom-action](custom-action.png)

![custom-action-add-output](custom-action-add-output.png)

接著要把安裝步驟中輸入的值，可以讓後面的程式可以讀取，選到剛剛所加入的 Output 並開啟 **屬性視窗** ，將欄位的名稱用這樣的格式包裝 `value=[name]` 寫到 `CustomActionData` 中。

![custom-action-properties](custom-action-properties.png)

### 新增 Installer Class

設定完畢以後，若直接這樣安裝，會發現報錯，因為缺少了一個程式的檔案，接著回到來源的專案新增檔案 `Installer Class` 。

![custom-action-add-install](custom-action-add-install.png)

開啟剛新增的檔案，可以看到裡面的程式會是這樣的。

```csharp
[RunInstaller(true)]
public partial class Installer1 : System.Configuration.Install.Installer
{
    public Installer1()
    {
        InitializeComponent();
    }
}
```

剛剛是在 `Installer` 設定輸出，在這邊同樣要對應的寫方法，可以透過 override 的方式來增加方法。相關的作法可以參考 [MSDN 的文件](https://docs.microsoft.com/zh-tw/dotnet/api/system.configuration.install.installer?view=netframework-4.8)。

### 修改config

先透過 `Context.Parameters` 的方法來取得剛剛所設定在 `CustomActionData` 中的值，接著使用 `ExeConfigurationFileMap` 取得 config 並且回寫。

> System.Diagnostics.Debugger.Launch() 可以用來在安裝過程中偵錯，是一個好用的技巧。

```csharp
// 可以避免寫入config檔的時候失敗
// [System.Security.Permissions.SecurityPermission(System.Security.Permissions.SecurityAction.Demand)]	
public override void Install(IDictionary stateSaver)	
{	
    base.Install(stateSaver);	
    // 偵錯時可以取消註解使用
    //System.Diagnostics.Debugger.Launch();
    var value1 = Context.Parameters["value1"];	
    //尋找安裝路徑中的App.config檔
    var exeConfigurationFileMap = new ExeConfigurationFileMap();	

    exeConfigurationFileMap.ExeConfigFilename = Context.Parameters["assemblypath"] + ".config";	

    var config = ConfigurationManager.OpenMappedExeConfiguration(exeConfigurationFileMap, ConfigurationUserLevel.None);	

    //將參數值寫回App.config檔
    config.AppSettings.Seetings["Value1"].Value = value1;

    config.Save();	
}
```

# 使用數位簽章增加信任度

剛製作完的 msi 檔案，直接放在網站上讓人下載，會被瀏覽器認為是不信任的檔案，使用者必須經過重重關卡才能真正拿到安裝檔
瀏覽器怕使用者誤下載一個執行檔，導致電腦出問題，因此會在下載後檢查檔案是否包含合法的數位簽章。

![download-msi-without-code-sign](download-msi-without-code-sign.png)

為了避免這樣的問題，必須要加上數位簽章 (code sign)，讓瀏覽器認可。因為這邊是公司其他部門的同事處理，就直接讓大家看一下有數位簽章會是什麼樣子。

![msi-with-code-sign](msi-with-code-sign.png)

# 結論

微軟提供的這個專案很陽春，很適合一些簡單需求的人使用，若有要比較客製化的步驟，則不建議使用。也是透過這次的實作才知道，原來瀏覽器會檢查簽章的合法與否，來避免使用者暴露於風險之中。

# Reference

* [[Visual Studio\] 應用程式佈署大作戰 - 為Setup Project加入自訂的對話視窗以修改App.config的內容 | Ouch@點部落 - 點部落 (dotblogs.com.tw)](https://dotblogs.com.tw/ouch1978/2011/08/11/visualstudio-setup-project-customize-app-config)
* [如何透過 C# 類別庫讀取 Web.config 或 App.config 的參數設定值 | The Will Will Web (miniasp.com)](https://blog.miniasp.com/post/2015/11/23/How-Class-library-read-config-from-webconfig-or-appconfig-file)
* [如何將自訂參數傳入 Installer 類別的 Install 方法 ( Part 2 ) | The Will Will Web (miniasp.com)](https://blog.miniasp.com/post/2009/04/06/How-to-pass-property-from-command-line-to-Custom-Actions)
* [程式學習筆記: 幫自己的exe檔建立自己的數位簽章 (limitx5.blogspot.com)](https://limitx5.blogspot.com/2017/05/windows-exe.html)

