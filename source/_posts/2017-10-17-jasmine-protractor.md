---
title: 前端測試框架-Jasmine&Protractor介紹
date: 2017-10-17 22:23:44
categories:
- Front-end
- Test
tags:
- Test
- Jasmine
- Protractor
---

今天要來談談兩個前端的測試框架，可以被使用在任何的前端網站開發中，當然也被應用在angular-cli中，但今天的內容其實不會搭配angular來做說明，而是以可以被獨立使用的工具

<!--more-->

- Jasmine
- Protractor

# Jasmine

一套JS的測試framework，從2009年開始到現在，是一套非常老牌的框架，就直接來看看該怎麼使用

```shell
# install jasmine
npm install --save-dev jasmine
#Initialize Jasmine in your project
./node_modules/.bin/jasmine init
```

設定`package.json`

```json
"scripts": {
  "test": "jasmine"
}
```

接著產生一個測試檔案`spec/first.spec.js`

```javascript
describe('first test', () => {
    it('hello jasmine', () => {
        expect(true).toBe(true);
        expect(false).not.toBe(true);
    });
});
```

> npm test

執行完上面的指令，可以得到第一次的測試成功

- `describe` 將關連的規格做群組的功能(允許巢狀)
- `it` 描述規格內容
- `expect` 測試的實際值
- `toBe` 判斷是否與前面的實際值相同
- `not` 反向指令

上面的範例就是jasmine的固定格式，只是說我們要透過不同的Matchers來做為測試的手法，接著就來介紹常用的Matchers

## Machers

###  toEqual

```javascript
it("toEqual", function () {
    let a = 12;
    expect(a).toEqual(12);
});
```

### toBeLessThan/toBeGreaterThan

```javascript
it("toBeLessThan & toBeGreaterThan", function () {
    let pi = 3.1415926;
    let e = 2.78;
    expect(e).toBeLessThan(pi);
    expect(e).not.toBeGreaterThan(pi);
});
```

### toThrow/toThrowError

```javascript
it("toThrow & toThrowError", function () {
    var foo = function () {
        throw new TypeError("foo bar baz");
    };
    expect(foo).toThrow();
    expect(foo).toThrowError("foo bar baz");
    expect(foo).toThrowError(/bar/);
    expect(foo).toThrowError(TypeError);
    expect(foo).toThrowError(TypeError, "foo bar baz");
});
```

當然還有更多的Machers，也可以到[官網文件](https://jasmine.github.io/api/2.8/matchers.html)查看，接著來看蠻常用到的功能Global功能

## Global

### beforeEach/afterEach

每個it在跑的時候都是獨立的事件，如果有些動作是要在每個it都要跑一樣的，那就可以使用這個功能
例如：網站登入的動作

```javascript
describe("A spec using beforeEach and afterEach", function () {
    var foo = 0;
  
    beforeEach(function () {
        foo += 1;
    });
  
    afterEach(function () {
        foo = 0;
    });
  
    it("is just a function, so it can contain any code", function () {
        expect(foo).toEqual(1);
    });
  
    it("can have more than one expectation", function () {
        expect(foo).toEqual(1);
        expect(true).toEqual(true);
    });
});
```

### beforeAll/afterAll

類似於beforeEach的功能，但beforeAll只會在整個describe中執行一次

```javascript
describe("A spec using beforeAll and afterAll", function () {
    var foo = 0;
  
    beforeAll(function () {
        foo += 1;
    });
  
    afterAll(function () {
        foo = 0;
    });
  
    it("sets the initial value of foo before specs run", function () {
        expect(foo).toEqual(1);
    });
  
    it("does not reset foo between specs", function () {
        expect(foo).toEqual(1);
    });
});
```

### xit/xdescribe

disable it/describe

### fit/fdescribe

只執行it/describe

## Spies

在測試中，mock的手法蠻常用到(在jasmine中是叫做spy)，尤其是要呼叫api或是使用第三方套件時，那些function我們認為是正確的，所以不希望把那些包進來測試，就做模擬的方式取代掉真實的呼叫，專注於我們自己所要測試的功能上

```javascript
describe("A spy", function() {
    var foo, bar = null;
 
    beforeEach(function() {
        foo = {
            setBar: function(value) {
                bar = value;
            }
        };
 
        spyOn(foo, 'setBar');
 
        foo.setBar(123);
        foo.setBar(456, 'another param');
    });
 
    it("tracks that the spy was called", function() {
        expect(foo.setBar).toHaveBeenCalled();
    });
 
    it("tracks all the arguments of its calls", function() {
        expect(foo.setBar).toHaveBeenCalledWith(123);
        expect(foo.setBar).toHaveBeenCalledWith(456, 'another param');
    });
 
    it("stops all execution on a function", function() {
        expect(bar).toBeNull();
    });
});
```

> `toHaveBeenCalled` 檢查是否真的有被呼叫過，`toHaveBeenCalledWith()` 是否符合呼叫function的參數列表

透過`spyOn`將setBar這個function做模擬，因此在這段的測試中，bar的值依舊是null，這個作法是完全攔截，而且沒有任何回傳結果，但我們通常應該會有一些輸出結果，就可以採用`and.returnValue`或是`and.callFake`

```javascript
describe("A spy, when configured to fake a return value", function() {
    var foo, bar, fetchedBar;
 
    beforeEach(function() {
        foo = {
            setBar: function(value) {
                bar = value;
            },
            getBar: function() {
                return bar;
            },
            getBar1: function() {
                return bar;
            }
        };
 
        spyOn(foo, "getBar").and.returnValue(745);
        spyOn(foo, "getBar1").and.callFake(function() {
            return 1001;
        });
 
        foo.setBar(123);
        fetchedBar = foo.getBar();
    });

    it("should not effect other functions", function() {
        expect(bar).toEqual(123);
    });
 
    it("when called returns the requested value", function() {
        expect(fetchedBar).toEqual(745);
    });
    
    it("when called returns the requested value", function() {
        expect(foo.getBar1()).toEqual(1001);
    });
});
```

# Protractor

由angular官方出的一套end-to-end(e2e)的framework，測試期間會開啟一個瀏覽器去執行所要測試的項目，如果是使用angular-cli所產生的專案，預設就已經包含，設定放在根目錄底下的`protractor.conf.js`

但今天說好不管angular-cli的，因此我們要自己安裝

```shell
npm install -g protractor
webdriver-manager update
webdriver-manager start
#http://localhost:4444/wd/hub
```

接著必須要有一個設定檔`conf.js`，並且寫好測試檔`spec.js`

```javascript
//conf.js
exports.config = {
  framework: 'jasmine',
  seleniumAddress: 'http://localhost:4444/wd/hub',
  specs: ['spec.js']//測試規格檔
}
```

`framework` 在這邊我使用了jasmine，但如果有其他習慣的框架也是能夠使用的

- Mocha
- Cucumber
- Serenity/JS

```javascript
// spec.js
describe('Protractor Demo App', function() {
  it('should have a title', function() {
    browser.get('http://juliemr.github.io/protractor-demo/');

    expect(browser.getTitle()).toEqual('Super Calculator');
  });
});
```

> protractor conf.js

好的，做完上面的動作，已經得到第一次的測試，也可以看到protractor開啟了網頁，並且拿到網頁上的內容做測試，這就是這套工具所提供的功能!

- `browser.get` 載入網頁
- `browser.getTitle` 取得網站title

到這邊做過第一次測試，但這種不是我們平常會做的測試，因此我們拿上面的網站，來進行內容的測試，首先要取得畫面中的輸入框，並且填入數字

```javascript
describe('Protractor Demo App', function () {
    beforeEach(() => {
        browser.get('http://juliemr.github.io/protractor-demo/');
    })
    it('first input', () => {
        let first = element(by.css('.input-small'));
        first.sendKeys('2');
        expect(first.getAttribute('value')).toEqual('2');
    });
});
```

> `sendKeys('2')` 模擬鍵盤並且送出2，`getAttribute('value')` 取得value的屬性

顯然我們只取得第一個輸入框，但畫面中有兩個輸入框，接著調整一下作法

```javascript
it('two input', () => {
    let els = element.all(by.css('.input-small'));
    let first = els.get(0);
    // let first = els.first(); same as get(0)
    first.sendKeys('2');
    let sec = els.get(1);
    // let sec = els.last(); same as get(1)
    sec.sendKeys('4');
});
```

> `element.all` 取得所有符合條件的element，`get(0)`、`first()`、`last()` 取得指定的element

填入兩個輸入框的值後，就可以按下按鈕，並且取得結果

```javascript
it('calculate', () => {
    let els = element.all(by.css('.input-small'));
    let first = els.get(0);
    // let first = els.first(); same as get(0)
    first.sendKeys('2');
    let sec = els.get(1);
    // let sec = els.last();
    sec.sendKeys('4');

    element(by.id('gobutton')).click();
    expect($('h2').getText()).toEqual('6');
})
```

> `by.id('gobutton')` id='gobutton'的選擇器，`click()` 產生click行為，`$('h2')` 使用jquery的寫法取得特定tag name的element，`getText()` 取得element的內文

執行完上面的測試案例，結果跟預期的一樣，恭喜測試完成！🥂
接著回過頭來看一下在protractor中比較重要的幾個物件

- `browser` 透過WebDriver操作瀏覽器，像是開啟網頁
- `element` 存取網頁的element
- `by` 選擇器

今天的demo程式[github連結](https://github.com/jiaming0708/testTool)

# 參考

* [angular test guide](https://angular.io/guide/testing)
* [jasmine](https://jasmine.github.io/)
* [protractor](http://www.protractortest.org/#/)
