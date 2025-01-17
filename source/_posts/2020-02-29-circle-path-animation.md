---
title: '[CSS] 利用animation做圓形的移動'
date: 2020-02-29 10:58:30
updated: 2020-02-29 10:58:30
categories:
- Frontend
- CSS
tags:
- CSS
---

在畫面上的一個物件的移動方式希望能夠是圓形的，除了用javascript計算的方式以外，有沒有css的方式能做到呢？今天要來一探究竟

<!-- more -->

# position模擬

不知道你會不會跟我第一時間的想法一樣，用兩點移動近似圓形的走法可不可以呢？使用 `animation` 搭配 `position` 的變化，理論上應該是可以做到這件事情

```css
@keyframes move{
  0%{
    left: 10px;
    top: 10px;
  }
  50%{
    left: 20px;
    top: 20px;
  }
  100%{
    left: 10px;
    top: 10px;
  }
}
```

但因為是兩點的移動是直線，這就等於要切非常細才會趨近於圓，可以想像一下...要算多久才能寫好一個圓形的移動？

# 使用rotate

另外一個方法就是使用 `rotate` 可以更輕鬆的達成目標，首先來看一下[MDN的定義](https://developer.mozilla.org/en-US/docs/Web/CSS/transform-function/rotate)

>The **`rotate()`** [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) function defines a transformation that rotates an element around a fixed point on the 2D plane, without deforming it

從定義中可以知道，`rotate` 會做角度的旋轉

## 原地旋轉

因此先寫個來測試一下旋轉這件事情，設定一個 `keyframe` 角度是從0到360

```css
@keyframes move{
  0%{
    transform: rotate(0deg);
  }
  100%{
    transform: rotate(360deg);
  }
}
```

{% codepen jiaming0708 dyoWBJw dark [result] %}

## 加上移動距離

現在已經有了旋轉的效果，那要讓物件有圓形的移動感，這時候可以使用 `translate` 來加上位移量

> 要記得只能有一個 `transform`，儘管 `translate` 沒有變化還是要每次都寫

```css
@keyframes move{
  0%{
    transform: rotate(0deg) translateX(10px);
  }
  100%{
    transform: rotate(360deg) translateX(10px);
  }
}
```

{% codepen jiaming0708 KKpmjRW dark [result] %}

## 維持正面

其實這樣已經完成了八成了，但還差一點點，就是希望字不要跟著轉，這時候要用到 `transform` 的特性

* 同一個function會累加計算
* 由左至右計算

### 累加計算

那我們先來測試第一件事情，**同一個function會累加計算**，寫 `rotate` 一個*0*和一個*45*，最後結果會是 *45*

```css
div{
	transform: rotate(0deg) rotate(45deg);
}
```

{% codepen jiaming0708 QWbvXVd dark [result] %}

### 由左至右

`animation` 會根據數字的不同來做動畫的效果，根據上面的作法，如果 `rotate` 先給一個數字接著減去同樣的數字，是不是就能夠移動了呢？

```css
@keyframes move{
  0%{
    transform: rotate(0deg) rotate(-0deg) translateX(10px);
  }
  100%{
    transform: rotate(360deg) rotate(-360deg) translateX(10px);
  }
}
```

答案是不行的，當 `rotate` 給了一個數值接著馬上減掉，這時候並不會讓 `animation` 認為這個function有變化量，因此再兩個 `rotate` 中間，必須要加上其他的 `function` 來做為緩衝，讓 `animation` 誤以為 `rotate` 有變化

### 組合結果

```css
@keyframes move{
  0%{
    transform: rotate(0deg) translateX(10px) rotate(-0deg);
  }
  100%{
    transform: rotate(360deg) translateX(10px) rotate(-360deg);
  }
}
```

{% codepen jiaming0708 KKpmjeW dark [result] %}

# 結論

這個作法一開始看到的時候，其實我根本無法理解原理，因為 `rotate` 算出來就是0阿，但為什麼他能夠引發角度的變化呢，原來是還要加上另一個特性，才能夠做出這樣的效果

另外，有興趣的話也可以加上 `scale` 或是將 `translate` 的值改變，試試看效果是什麼喔！

# 參考

* [Animating Circular Paths Using CSS3 Animations](https://www.useragentman.com/blog/2013/03/03/animating-circular-paths-using-css3-transitions/)

* [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/transform-function)