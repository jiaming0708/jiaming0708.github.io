---
title: react component重構心得分享
date: 2019-03-12 20:25:23
updated: 2019-03-12 20:25:23
categories:
- Frontend
- React
tags:
- React
---

最近在寫新的功能時，因為有些跟舊有的功能有關係，所以一定會去改到原本的code，沒去看都不知道，看了大罵WTF，這也太噁心了吧...

寫過react的人都知道，component分為兩種，一個是class component，而另一個就是pure component，不管用哪一種，不知道各位可以接受一個js(x)的行數有多少，或者更精準的是render幾行內你可以接受

<!-- more -->

先說說我可以接受的一個render element大概是50行內，但通常我自己寫是10行左右，我是覺得一個element如果超過太多，閱讀性整個會很差，不管是不是包含邏輯

# 可怕的component

先來看看底下這段，經過一點點的處理，但基本上架構是跟原本一樣

{% gist 9218dae11fca571f215a0173de9e09ad HugeComponent.jsx  %}

>  現在需求來了，要增加一個button在sidebar，請問你要花多久時間才能夠增加

# 重構

第一次看到的時候，完全不知道怎麼下手，所以我打算要做一件事情**重構**，先把element整理的比較好懂一點，就讓我們來動作試試看吧

## 先抽變數後的

看到程式中，有一些element是根據變數來決定要不要render，這種的最好被拉出來，因此我們先針對這種來做

```jsx
showCancelPopup && (
  <Modal
    onClose={this.closePopup}
  >
      {/* ... */}
  </Modal>
)
```

把上面的這種element變成獨立的function，讓主render可以看起來輕鬆一點，從原本的190行變成90行

> [完整的可以看這邊](https://gist.github.com/jiaming0708/9218dae11fca571f215a0173de9e09ad#file-hugestep1-jsx)

{% gist 9218dae11fca571f215a0173de9e09ad HugeStep1OnlyRender.jsx  %}

## 區塊抽離

做完第一步以後，覺得有舒服一點，但還是不夠好，閱讀了一下整個結構，發現其實是可以分為幾個區塊，接下來就針對這些來動個手術

把兩個最大的拆出去以後，主render就只剩下20行，整個爽度都上來了

> [完整的可以看這邊](https://gist.github.com/jiaming0708/9218dae11fca571f215a0173de9e09ad#file-hugestep2-jsx)

{% gist 9218dae11fca571f215a0173de9e09ad HugeStep2OnlyRender.jsx  %}

## component化

做完前面兩個步驟，已經可以先交卷，為什麼會這樣說，主力的時間應該都在feature上面，重構只是順帶一做，當然如果有足夠的時間一定要把這步驟做完，整個才會夠乾淨

要做這步驟前一定是前面兩個做完，才有機會做到，至於這個component有沒有機會被共用這又是另一件事情了，這邊不討論

接著來說一下我怎麼規劃component，主要考量會是這兩個部分

### 資料的來源

理論上我們會希望這是一個pure component，所以資料都是由外面傳進來，但會不會有那種資料反而是去redux中取得會比較好？例如說資料只有這個component用，結果已經傳遞了n層

### 事件的觸發

是把事件往外傳，還是直接由component呼叫redux的action執行？

### 範例

講了那麼多，就來實作一下，把`renderCancelPopup`變成component，至於為什麼我會拿這個，因為這個行為最為單純，最適合第一時間重構

總共有三個事件，那我們都讓事件往外傳，至於文字要不要是外面傳的就看你囉，基本上非共用的我比較不會從外面傳，除非真的超級少

```jsx
function CancelPopup({
    t,
    closePopup,
    cancelQ,
}){
    return (
        <Modal
          onClose={closePopup}
        >
          <SimpleDialog
            title={t('')}
            theme={dialogTheme}
          >
            <Button
              theme={gray}
              ghost
              onClick={closePopup}
              text={t('')}
            />
            <Button
              theme={pink}
              onClick={cancelQ}
              text={t('')}
            />
          </SimpleDialog>
        </Modal>
    );
}
```

在render就可以變成這樣

```jsx
{showCancelPopup && <CancelPopup closePopup={closePopup} cancelQ={cancelQ} />}
```

# 結論

在一開始寫的時候我也傾向先不拆分，因為那時候這個component的拆法和行為整個都還不是非常確定，而等到整個行為比較確定以後，再來開始進行重構的動作，讓自己好閱讀以外，也讓review你的code的人不會腦袋打結

這個功能的重構，我目前只有做到第二步以後就交卷了，要等下個sprint稍微有空一點的時候才有辦法來處理

>  重構是必經的路程，適時的重構可以讓心情愉悅，不用鬱悶的寫到吐血