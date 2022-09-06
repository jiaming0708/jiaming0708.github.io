---
title: '[CSS] 在flex呈現省略號'
date: 2019-04-16 11:10:41
categories:
- Frontend
- CSS
tags:
- CSS
---

最近做一些新的畫面，其中有一個問題很妙，在一般的情況下要呈現省略號，其實算是蠻容易的，但在flex中卻有點麻煩，今天要來探討一下這個部分

<!--more-->

# 畫面架構

來看一下畫面需求是這樣，左邊有一個清單，每一張卡片裡面還分兩塊，右邊的資訊過長的時候，必須是需要呈顯省略號

![layout](layout.png)

# 討論省略號

要能夠呈現省略號，必須要達成三個條件，少掉一個其實都無法看到省略號

* width
* overflow
* text-overflow

後面兩個其實非常容易達成，但是width呢，在flex中該怎麼讓width固定，尤其是按照比例的那種

# 程式撰寫

## html結構

這個需求的重點是在左邊的list，因此右邊就簡單做就好，在這邊可以看到在一張card裡面會有所謂的圖片和內容區在左右兩邊，

```html
<div class="root">
  <div class="list">
    <!-- 重點區塊 -->
    <div class="card">
      <div class="pic"></div>
      <div class="content">
        <div class="title"></div>
        <div class="descr"></div>
      </div>
    </div>
  </div>
  <div class="body">
    Body
  </div>
</div>
```

## css

接下來來看一下css的部份，先用最直覺的寫法，結果發現怎麼都超過了阿!!

```scss
.root{
  display: flex;
  border: 1px solid black;
  height: 500px;
  
  .list{
    width: 30%;
    
    .card{
      display: flex;
      border: 1px solid blue;
      padding: 1rem;
      
      .pic{
        width: 100px;
        border: 1px solid green;
      }
      // 重點區塊
      .content{
        flex: 1;
        margin-left: 0.5rem;
        
        .title{
          font-weight: bold;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
        
        .descr{
          margin-top: 0.25rem;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
      }
    }
  }
  
  .body{
    flex: 1;
    text-align: center;
  }
}
```

![oveflow](overflow.png)

怎麼完全width完全沒有被控制住呢，說好的省略號呢！

# 解法

那先試著給width一個固定值，省略號的確可以work，跟上面所講的三要素是相同，但給固定值絕對不是我們需要的，就讓我們一起試著找一些方法來解決吧

## overflow:hidden

> 這應該算是比較合理的解法

一定會有人覺得奇怪，在 `.descr` 和 `.title` 已經有給，但為什麼還要在給呢？
這邊要增加的是設定在 `.content` ，因為已經設定了 `flex:1` ，這時候寬度已經被分配剩餘的空間，那我們只要加上這個屬性，就能夠正常顯示省略號

```scss
.content{
  flex: 1;
  margin-left: 0.5rem;
  // 重點
  overflow: hidden;
  
  .title{
    font-weight: bold;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  
  .descr{
    margin-top: 0.25rem;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
}
```

## min-width:0

> 小心服用，而且不好解釋給同事聽orz

這是一個[查來的作法](https://css-tricks.com/flexbox-truncated-text/)，其實我也沒有很懂他的概念，也許有人比較清楚可以在分享分享，但一樣可以達到目的，讓寬度限制住，一個黑魔法來著...

```scss
.content{
  flex: 1;
  margin-left: 0.5rem;
  // 重點
  min-width: 0;
  
  .title{
    font-weight: bold;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  
  .descr{
    margin-top: 0.25rem;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
}
```

# 參考

* [codepen](https://codepen.io/jiaming0708/pen/vMWaeP)

* [flexbox-truncated-text](https://css-tricks.com/flexbox-truncated-text/)