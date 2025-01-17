---
title: 如何在 cli 快速操作 json 資料 - jq
date: 2022-06-04 21:47:46
updated: 2022-06-04 21:47:46
categories:
- shell
tags:
- jq
- shell
---

想在 cli 的環境下要對 json 的檔案做修改，如果要用一行一行讀取後處理，實在是太麻煩而且太容易改錯，還好有一套工具 [jq](https://stedolan.github.io/jq/) ，可以快速的操作整個 json 資料。
我把這個應用在 CI/CD 的環節中，在當下調整一些設定檔。

> jq 到哪都是強者!!!

<!-- more -->

jq 可以被使用在任何一個有 cli 的 os 上 (Linux, OS X and Windows...etc.)，選取的方法是透過 filter 來進行操作，先用個簡單的範例來示範，如何取得 foo 的值。

> 好像看到 $jq 的影子?

```bash
echo '{"foo": 42, "bar": "less interesting data"}' | jq '.foo'
> 42
```

# linux 安裝

接下來將會起一個 Debian 的容器來做為示範，如何安裝並且操作。

> 不想安裝只是想嘗試的話可以用官方提供的線上工具 [jq play](https://jqplay.org/)

```bash
docker run --rm -it debian
```

進入到容器的環境內後，開始執行安裝，記得先 update 確保安裝的時候找的到套件

```bash
apt-get update
apt-get install -y jq
```

# 認識 filter

主要會專注在介紹最後實務上要用到的功能。

## 根節點

json 資料的格式都會有個最上層，也就是所謂的根結點，不管是陣列或是物件的形式，都是從最外層開始往內找。

```bash
echo '{"foo": 42, "bar": "less interesting data"}' | jq '.'
> {
>  "foo": 42,
>  "bar": "less interesting data"
> }
```

## 物件操作

```bash
# 單一屬性
echo '{"foo": 42, "bar": "less interesting data"}' | jq '.foo'
> 42
# 屬性不存在回 null
echo '{"notfoo": true, "alsonotfoo": false}' | jq '.foo'
> null
# 多個屬性
echo '{"foo": 42, "bar": "less interesting data"}' | jq '.["foo", "bbb"]'
> 42
> null
```

## 陣列操作

```bash
# 取得整個陣列並轉成 iterator
echo '[{"name":"JSON", "good":true}, {"name":"XML", "good":false}]' | jq '.[]'
> {"name":"JSON", "good":true}
> {"name":"XML", "good":false}
# 指定陣列位置
echo '[{"name":"JSON", "good":true}, {"name":"XML", "good":false}]' | jq '.[0]'
> {"name":"JSON", "good":true}
# 切割陣列 (也可用在字串中) [start:end]
echo '["a","b","c","d","e"]' | jq '.[2:4]'
> ["c", "d"]
```

## 資料運算

`+` 和 `-` 可以用來運算數值或是陣列/字串的串接

```bash
# 運算數值
echo '{"foo": 42, "bar": "less interesting data"}' | jq '.foo + 3'
> 45
echo '{"foo": 42, "bar": "less interesting data"}' | jq '.foo - 2'
> 40
# 陣列串接
echo '["a","b","c","d","e"]' | jq '. + ["f","g"]'
> ["a","b","c","d","e","f","g"]
echo '["a","b","c","d","e"]' | jq '. - ["a"]'
> ["b","c","d","e"]
# 字串串接
echo '{"foo": 42, "bar": "less interesting data"}' | jq '.bar + " concat another string"'
> "less interesting data concat another string"
```

## 修改內容

```bash
echo '{"foo": 42, "bar": "less interesting data"}' | jq '.foo = 44'
> {"foo": 44, "bar": "less interesting data"}
```

## 讀取檔案

```bash
echo '{"foo": 42, "bar": "less interesting data"}' > file.json
jq '.foo' file.json
> 42
```

## 傳入參數

```bash
# 非 json 格式
echo '{"foo": 42, "bar": "less interesting data"}' | jq --arg concat " pass from arg" '.bar + $concat'
> "less interesting data pass from arg"
# json 格式
echo '["a","b","c","d","e"]' | jq --argjson concat '["f","g"]' '. + $concat'
> ["a","b","c","d","e","f","g"]
```

# 實際修改

實際上真正需要的是調整 db.json 這個檔案，先來看看資料格式。

```json
{
  "DBConfig": {
    "DefaultDB": "LocalOrcl",
    "EnableLog": true,
    "DoNotCompleteFlag": false,
    "DataBases": [
      {
        "DBName": "LocalOrcl",
        "ConnectionString": "Data Source=localhost/ORCL;USER ID=jimmy;PASSWORD=jimmy;",
        "Provider": "Oracle"
      }
    ]
  }
}
```

目標是要修改 DefaultDB 的值，並且增加一組連線資訊，基本上就是把前面所提到的 filter 都用過一次。

```bash
CONNECTION='{"DBName":"CI","ConnectionString":"Data Source=localhost/ORCL;USER ID=CI;PASSWORD=CI;","Provider":"Oracle"}'
jq '.DBConfig.DefaultDB = "CI"' db.json > tmp.$$.json && mv tmp.$$.json db.json
jq --arg CONNECTION "$CONNECTION" '.DBConfig.DataBases += $CONNECTION' db.json > tmp.$$.json && mv tmp.$$.json db.json
```

就可以看到 db.json 的內容被改成以下。

```json
{
  "DBConfig": {
    "DefaultDB": "CI",
    "EnableLog": true,
    "DoNotCompleteFlag": false,
    "DataBases": [
      {
        "DBName": "LocalOrcl",
        "ConnectionString": "Data Source=localhost/ORCL;USER ID=jimmy;PASSWORD=jimmy;",
        "Provider": "Oracle"
      },
      {
        "DBName": "CI",
        "ConnectionString": "Data Source=localhost/ORCL;USER ID=CI;PASSWORD=CI;",
        "Provider": "Oracle"
      }
    ]
  }
}
```

# Reference

- [jq Manual (development version) (stedolan.github.io)](https://stedolan.github.io/jq/manual/)
