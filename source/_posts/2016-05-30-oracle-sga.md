---
title: 記憶體控制sga
date: 2016-05-30 22:29:03
categories:
- Back-end
- Oracle
tags:
- oracle
---

最近在玩系統負載測試，PM說用到最新的那台server上跑看看，於是就把系統丟上去開始測試
沒跑不知道，測試下去才發現，怎麼記憶體吃了96%
很怕是自己的系統導致，所以趕快的確認一下是誰吃了那麼多

<!--more-->

還好兇手不是我(乎)，是三個DB，兩個oracle各吃15G，一個MS SQL吃13G（server共47G）
問了一下高手看看這種是什麼情況，原來是sga在搞鬼

先下個指令確認一下，確認現在設定多少

``` sql
show sga;
show parameter sga_target
show parameter sga_max_size;
```

果然是target和max_size設定一樣導致，於是將sga_target先調小，sga_max_size不變避免未來需要吃太多

``` sql
alter system set sga_target=8192M;
```
