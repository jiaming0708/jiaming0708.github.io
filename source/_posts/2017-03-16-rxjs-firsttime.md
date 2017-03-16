---
title: RX初體驗(?
date: 2017-03-16 22:10:52
categories:
- Front-end
- RxJS
tags:
- RxJS
---

## 前情提要
其實也不算是初體驗啦
因為之前就有把promise的程式改成用rx，但那時候還是非常不了解

最近在跟人一起寫angular的workshop，飲料維護及訂購的功能
就開始用一些之前沒用到的功能rx、form control等...

嗯～還是不要太多廢話XD

## 從promise改成rx
從ng1的時候，就都是寫promise，所以剛轉到二的時候，看到官網寫rx，就沒有想要去寫他o.0（年少不懂事...
一開始的promise
``` typescript
getStepListByProjectId(id: number) {
   return this._http.post(this.url + "GetStepListByProjectId", { projectId: id })
       .toPromise()
       .then(res => {
           var data = res.json();
           if (data.Result) {
               //轉換為前端使用物件
               data.StepList = this.parseStepDataList(data.StepList);
               //預設值第一個選取
               if (data.StepList.length > 0) {
                   data.StepList[0].SelectedFlag = true;
               }
           
           return data;
       })
       .catch(this.handleError);
}
```

改寫成rx後，在重新整理過，是不是感覺更精簡了
但這時候其實還沒什麼感覺，因為比較算是轉換而已，對於rx還是很不熟...
``` typescript
getStepListByProjectId(id: number) {
   return this._http.post(this.url + "GetStepListByProjectId", { projectId: id })
        .switchMap(this.checkResult)
        .map(this.parseStepDataList)
        .catch(this.handleError);
}

private checkResult(res: any): Observable<any> {
    var data = res.json();
    if (!data.Result) {
        Observable.throw(data.ErrorMessage);
    }
    return Observable.of(data.StepList);
}

```

## rx進一步的應用

### concatMap、toArray
這次的需求是這樣的，從api取得的資料中，沒有其中一個物件中的欄位state
所以我要把取得的資料重新assign欄位值

這樣的寫法其實已經可以達到我的需求，但總覺得不是很好看
這是第一版
``` typescript
getBeverageList(): Observable<BeverageData[]> {
  return this._http.get(this.beverageAPI, this.options)
    .map(response => response.json() as BeverageData[])
    .map(item => item.map(p => {
      p.state = StateType.None;
      return p;
    }))
    .catch(this.handleError);
}
```

剛好kevin介紹了一個工具[Quokka.js](https://quokkajs.com/docs/)，可以不用console.log就可以把結果呈現在旁邊
於是就開始實驗怎麼寫，可以讓這段更好看
先把陣列拆開，做完以後再組合起來，這樣的寫法看起來是不是更清楚一點
這是第二版
``` typescript
getBeverageList(): Observable<BeverageData[]> {
  return this._http.get(this.beverageAPI, this.options)
    .map(response => response.json() as BeverageData[])
    .concatMap(p => p)
    .map(p => {
      p.state = StateType.None;
      return p;
    })
    .toArray()
    .catch(this.handleError);
}

```

### mergeMap
post資料回server端後，想要重新取得全部的資料，這時候又不希望由前面再次呼叫
這時候可以直接在service層一次做完

那因為是呼叫另一個已經寫好的service，所以必須要把自己的observable和另一個接起來
這時候我們要使用到mergeMap

``` typescript
postBeverage(index:number,  beverage: BeverageData):Observable<BeverageData[]> {
  return this._http.post(this.beverageAPI + "/" + index, beverage, this.options)
    .mergeMap(resp => this.getBeverageList())
    .catch(this.handleError);
}
```

## 後續
rx這東西真的要有案例來學，才有辦法體會他的奧妙，不然看文件看多了還是不太清楚如何搭配使用