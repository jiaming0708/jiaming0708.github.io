---
title: '[Debug] AjaxToolKit Calendar的時間問題'
date: 2023-01-05 11:49:02
updated: 2023-01-05 11:49:02
categories:
- Webform
tags:
- Webform
- ASP.NET
- Debug
thumbnail:
---

在一個 Webform 的專案中，使用到 AjaxToolKit 的元件 `CalendarExtender`來選擇日期，設定的時間格式為 `2023/01/05  23:59:59`，在瀏覽器做日期的選擇後，會變成 `2023/01/05 23:59:00`，也沒有看到任何地方有去變更時間，同事覺得奇怪，一起追查問題以後發現這是一個 bug。

<!-- more -->

## 問題追查

專案中的程式是這樣寫，`ttbEndDate` 的值是給今天的 23:59:59，沒有去接任何事件。

```html
<asp:TextBox ID="ttbEndDate" runat="server" CssClass="CSInput" Width="90%"></asp:TextBox>
<ajaxToolkit:CalendarExtender ID="ttbEndDate_CalendarExtender" runat="server" TargetControlID="ttbEndDate"
    PopupButtonID="ibtEndDate" Format="yyyy/MM/dd HH:mm:ss">
</ajaxToolkit:CalendarExtender>
<asp:ImageButton ID="ibtEndDate" runat="server" ImageUrl="../../Images/Calendar.JPG"
    Height="20px" />
```

這是 ajax 的元件，那就試著從 javascript 開始找起，開啟網頁的開發者工具，html 的內容中，搜尋 `ttbEndDate_CalendarExtender` 可以找到下面這段程式，確定的是這段只有做初始化，沒有塞入值之類的行為。

```javascript
Sys.Application.add_init(function() {
    $create(Sys.Extended.UI.CalendarBehavior, {"button":$get("ibtEndDate"),"format":"yyyy/MM/dd HH:mm:ss","id":"ttbEndDate_CalendarExtender"}, null, null, $get("ttbEndDate"));
});
```

前面是從 html 去看原始碼的宣告，接著就要切換到來源找到 `CalendarBehavior` 的運作，開啟網頁的 aspx 可以看到裡面有滿滿的 js 宣告，可以從註解中找到 `CalendarBehavior` 的區塊。

> 這種壓縮過的程式會比較難閱讀，可以點下方的 `{}` 圖示來還原比較好讀的排版。參考 [開發不難，會 Debug 就好！如何靈活運用 Chrome DevTools 來開發網站 | 五倍紅寶石・專業程式教育 (5xruby.tw)](https://5xruby.tw/posts/how-to-use-chrome-devtools)

![source-aspx-js](source-aspx-js.png)

看展開後的排版，從上到下的把一段大概看一下，猜測寫入值的部分應該是在 `set_selectedDate`，從這邊下個中斷點，開始一行一行的看邏輯。
![calendarbehavior.png](calendarbehavior.png)

`d` 是選擇到的日期，`f` 是原本選擇的日期也就是帶有時分秒的，在 4734 行開始，會把 `f` 的時分秒開始寫入到 `d` 身上，問題就在這邊，有 `時Hour`、`分Minute`、`毫秒Millisecond`，但是沒看到 `秒Second`。
嘗試把這行程式在 `監看` 執行，可以看到 `ttbEndDate` 的值就是正確的了!!

```javascript
d.setUTCSeconds(e.getUTCSeconds());
```

## 解決方法

上面雖然找到問題了，但那個是元件中所產生的 js，沒辦法直接去改動到，先檢查專案 AjaxToolKit 的版本為 `4.0.60919` ，在 nuget 中最新的版本為 `20.1.0`，在 github 上面有找到 [source code](https://github.com/DevExpress/AjaxControlToolkit/blob/master/AjaxControlToolkit/Scripts/Calendar.js)，在最新版本中已經有被修正。

> 這個問題一直到 `15.1.4` 才被修正

同事是採用不升級的手法來處理，寫一個 js 的方法，把後面的時間固定。

```javascript
function endDateSelect(ev) {
    var calendarBehavior1 = $find("ttbEndDate_CalendarExtender");
    var d = calendarBehavior1._selectedDate;            
    calendarBehavior1.get_element().value = d.format("yyyy/MM/dd") + " 23:59:59";
}
```

在 `CalendarExtender` 的屬性宣告`OnClientDateSelectionChanged`，讓元件可以去呼叫剛剛所寫好的方法。

```html
<ajaxToolkit:CalendarExtender ID="ttbEndDate_CalendarExtender" runat="server" TargetControlID="ttbEndDate"
    PopupButtonID="ibtEndDate" Format="yyyy/MM/dd HH:mm:ss" OnClientDateSelectionChanged="endDateSelect" >
</ajaxToolkit:CalendarExtender>
```

## 結論

雖然這是古老的技術，也是可以透過現代的工具來幫助我們更容易地找到問題，感謝 open source 的年代，可以更容易的去找到版本的變更，可惜的是版本的紀錄是從 15 開始，無法看到更早的變更，就不能那麼確認是否有 breaking change。
