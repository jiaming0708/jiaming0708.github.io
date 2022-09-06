---
title: '[GTM] pageview加上content group'
date: 2019-02-19 12:22:02
categories:
- Frontend
- GTM
tags:
- GTM
---

最近有一個需求是在特定頁面要加上content group的資料，很多的功能用GA都還蠻簡單，但是用了GTM就會變得比較複雜一點，今天來分享一下用GTM怎麼完成這個需求

<!-- more -->

#設定GA

最一開始當然就是要先把GA設定起來，這樣後續才有辦法做測試並且確認資料是否正確，首先contnet group是設定在`view`底下而不是`property`，所以要先到admin中找到`Content Grouping`的設定

![admin-view](admin-view.png)

接著點擊新增然後輸入Name和index，在規劃的時候也要注意一下content group只能有五組

> 這邊要注意新增後只能停用不能刪除

![create-content-grouping](create-content-grouping.png)

有注意到圖的最下面，有給一些範例要怎麼去寫入這個資料，但這個對於我們要搭配GTM的來說，其實是沒什麼幫助的，因此可以忽略不看

# 設定GTM

## Data Layer Variable

GTM有一個功能叫做**Data Layer Variable**，這個可以接收從web打上來的一些資料，但缺點是資料會一直保留到清除為止，因此除了DL以外還要設定一個**Custom Function**

* DL-TYPE(**Data Layer Variable**)
  名稱就給他`TYPE`，在程式我們只要給對應的名稱GTM就會對應到
  ![DL-TYPE](DL-TYPE.png)

* Get TYPE
  寫一個判斷式，當網址不是指定頁面(**post**)的時候要給其他值，避免掉資料殘留的問題，因為網站本身架構比較單純主要分兩個頁面，HOME、POST，所以只要簡單判斷並且給一個定值就好

  ```javascript
  function(){
  	var val = {{DL-TYPE}};
  	var path = {{Page Path}};
  	if(!/(\/post\/)/.test(path)){
        val = 'HOME';
      }
      
      return val;
  }
  ```

## GA Setting

接著就可以在`Google Analytics Settings`加上content group的設定，打開`More Setting`找到`Content Groups`新增一個index是1，值從`Get TYPE`這個function取得
![GA-setting](GA-setting.png)

# Send Data Layer

## push data layer

TYPE的資料是`componentDidMount`的時候透過API拿回來，因此我們可以在這邊再打出Data Layer資料

```javascript
window.dataLayer.push({
    'TYPE': 'VIDEO',
});
```

寫完使用preview的功能看一下有沒有資料，慘～怎麼都沒拿到，再仔細思考一下整個流程

> home -> (page view) -> post -> (send data layer)

從首頁到post頁面的時候，就已經會先打一個history，然後在GTM會發一個page view出去，然後post頁面會等到api回來的時候才打datalayer，這時候GTM已經把資料送出去根本不會包含datalayer

## event取代history

上面的狀況導致我們不能直接在history change就送page view，因此要改一下時機點，變成是由程式自己發一個事件，這時候在triggers新增一個`Custom Event`
![post-view-event](post-view-event.png)

當然也要過濾掉原本的history設定，從`All History Changes`改成`Some History Changes`，並且設定path不包含 `/post/`
![hisotry-event](hisotry-event.png)

整個設定完以後，程式也在`componentDidMount`發送event，就可以用preview看看結果是不是如預期！

# 結論

這個功能其實很簡單，官方也有說明怎麼設定，但因為時間點的問題，導致要多做一些動作，才能達到原本的預期，如果說你只是一個url的值當做content group，就完全可以忽略上面所講的這些