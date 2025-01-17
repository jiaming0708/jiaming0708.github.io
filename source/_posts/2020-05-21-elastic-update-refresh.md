---
title: 在 elasticsearch 更新後立即能夠查詢
date: 2020-05-21 22:24:59
updated: 2020-05-21 22:24:59
categories:
- Backend
- ElasticSearch
tags:
- ElasticSearch
- Database
---

一般的關聯式資料庫下 update 以後，如果沒有下 commit 大部分情況是拿到舊的資料，但是在 elastic search 的環境下，沒有所謂的 commit，在異動資料後馬上執行 search 卻拿不到更新後的結果。

<!-- more -->

只要你在下指令的時候，加上 **refresh** 這個條件並且指定為 true，就可以馬上讓 search 看到異動後的資料。

> 官方不太建議使用，如果真的要用要小心，[官方文件](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)

這邊附上 elasticsearchjs 的範例，將符合的 id 都改為 deleted 的狀態。

```javascript
elastic.updateByQuery({
  index: TABLE_NAME,
  refresh: true,
  body: {
    script: {
      lang: 'painless',
      source: 'ctx._source["deleted"] = true',
    },
    query: {
      terms: {
        id: ids,
      },
    },
  },
});
```