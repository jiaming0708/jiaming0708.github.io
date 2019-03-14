---
title: postgres對於json的支援度
date: 2019-03-14 21:47:58
categories:
- Back-end
- Postgres
tags:
- Postgres
---

data team的同事回報了一個bug，是我寫入的資料格式不正確，導致他們在解析資料上的有困難，先把寫入資料的問題解決以後，要幫忙把錯誤的資料導正回來，才發現postgres對於json的支援度蠻高

<!-- more -->

# 緣由

先來說說這種資料格式怎麼產生的，我們有一個academy的網站，app會用web view的方式開啟，為了要追蹤這是由哪邊連近來的，所以app在開網頁的時候會加上query parameter給我們，並且我們將這些資訊寫入到db作為觀看記錄

格式大概會是這樣 `?region=tw&platform=android`，但是就出現了這種情況`?region=tw&platform=android&region=tw&patform=android`，收到了雙份的參數，導致寫入的時候變成了錯誤的格式

寫入的資料變成這樣

|      | region      | platform              |
| ---- | ----------- | --------------------- |
| 正常 | tw          | android               |
| 錯誤 | {'tw','tw'} | {'android','android'} |

# 解決方法

這個資料格式第一眼看到，就是往json的方向去找，剛好查到postgres[官方文件](https://www.postgresql.org/docs/9.3/functions-json.html)表示從**9.3**的版本以後就可以直接使用

因此就馬上很開心的使用了裡面的一個function叫做`json_object_keys`是用來拿到json物件的key值，沒想到收到錯誤是傳進去的值不對，馬上試著用另一個看看`to_json`，沒想到一樣收到錯誤說這個資料格式不正確

原來這個根本不是json物件，而是一個不知道什麼的資料格式，那該怎麼辦呢

## 資料轉型

幸好，還有另一種轉型的function叫做`array_to_json(anyarray [, pretty_bool])`，我們可以這樣使用

```sql
select array_to_json(region::text[]) from table1
-- ['tw', 'tw']
```

並且得到一個正確的json陣列，接著就能根據這樣的資料透過json function做解析

## 取值

在postgres有提供一種operator的寫法，來取得json物件的資料

```sql
select '{"tw":123}'::json->'tw'
-- 123
select '["tw", 123]'::json->0
-- "tw"
```

那我們就來試試看搭配我們的資料，沒錯啦，可以完美的拿到資料，但有沒有發現一個小缺點資料是`'tw'`帶著單引號的

```sql
select array_to_json(region::text[])::json->0 from table1
-- 'tw'
```

這時候要用另一個方法來拿到只有`tw`的資料，使用`>>`，真是太完美了！

```sql
select array_to_json(region::text[])::json->>0 from table1
-- tw
```

# 結論

在postgres的json相關功能提供的蠻齊全，並且也有支援json作為欄位格式，在使用上就更加的彈性

這次的意外順便學到了一個新的東西，順便碰一下很久沒有碰到的資料庫，趕快把技能點上來XD

# 參考

* [JSON Functions and Operators](https://www.postgresql.org/docs/9.3/functions-json.html)

