---
title: ReactNative初體驗
date: 2017-03-30 15:13:06
categories:
- Frontend
- ReactNative
tags:
- ReactNative
---
最近要和朋友一起寫個app，他們提出了要用react native
但完全沒有經驗，只好邊寫邊摸囉
分享一下下自己的一點點經驗囉

<!--more-->

## element
在native中，不能使用html中的element
而是要改使用Text、View、Image這些...
* Text其實就是span或是label
* View就是div
* Image就img
* TextInput也就是input囉

## css
native中我們一樣是使用css來作為樣式的設定
但寫法有點不一樣，是採用駝峰式的命名，數字以外的都要用單引號包起來
而且有的是有限制的，不是每個都能像是css那樣給值喔
還是要查一下官方文件，才能確定怎麼使用

``` css
backgroundColor: 'rgba(0, 0, 0, 0.6)'
margin: 2 /*number only*/
```

## if
在react中，要動態決定哪些內容要不要呈現
可以用if去判斷來撰寫，但在native中，這個語法會報錯?
因此可以用三元運算子(?:)來實作囉
``` react
this.props.data.publice_date ? (
    <View style={styles.dateBlock}>
        <Text style={styles.articleDate}>{this.props.data.publice_date + ' by ' + this.props.data.author}</Text>
    </View>
) : (
        <View style={styles.dateBlock}>
            <Text style={styles.eventDurtion}> {'日期:' + this.props.data.s_date + '~'this.props.data.e_date}</Text>
        </View>
    )
```

或是可以用更簡單的方法&&
``` react
this.props.data.category && (
    <View style={styles.categoryBlock}>
        <Text style={styles.category}>{this.props.data.town}</Text>
        <Text style={styles.category}>{this.props.data.category}</Text>
    </View>)
```

不管哪種方法，記得都要用()包起來，然後只能有一個root節點
也就是說不能像下面這樣
``` react
this.props.data.category && (
        <Text style={styles.category}>{this.props.data.town}</Text>
        <Text style={styles.category}>{this.props.data.category}</Text>)
```

## for
那如果說要用迴圈去把資料呈現出來，有兩個方法
react native有提供ListView，但這個只能row的方式呈現
如果需要往橫向發展的，千萬不能用

那我們要怎麼用勒，可以使用陣列自有的map這個operator
裡面有個key的屬性，記得要加，不然執行測試時也會看到他的提示
不過還不是很清楚這個功能的作用
``` react
this.props.data.tagList.map((tag, key) => {
    return <Text style={styles.tag} key={key}>{tag}</Text>
})
```

## 排版
在這邊你能夠盡情的使用flex來協助你排版
完全不用考慮到瀏覽器的問題v(￣︶￣)y
``` css
flex:1,
flexDirection: 'row'
```