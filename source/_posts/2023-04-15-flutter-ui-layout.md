---
title: '[Flutter] 排版應用介紹'
categories:
  - Flutter
tags:
  - Flutter
thumbnail: /images/flutter.png
date: 2023-04-15 23:13:05
---

作為一名主要開發平台為網頁的開發者，進入 Flutter 世界的開發體驗讓我發現元件結構是最需要適應的地方。在網頁開發中，我們可以透過 CSS 樣式來控制元素的排版，包括長寬、內外邊距，甚至彈性佈局 (flex) 或網格系統 (grid) 等。除非使用定位 (position)，否則所有元素都處於同一層級。然而在 app 中，每個元素都被當作不同的「層」來處理，因此為了達到類似網頁的效果，必須使用不同的元件進行組合。

<!-- more -->

## 長寬和內外邊距的控制

在 Flutter 中，長寬和內外邊距的控制可以透過不同的元件來實現。以下是兩種常見的元件：

### SizeBox

如果只需要設定元件的長寬，可以使用 SizeBox 元件。例如：

```dart
var comp = const SizedBox(
	height: 36,
	child: Text('我是文字')
);
```

> 若想讓元件自動擴展到 parent 的大小，可以使用 Expanded 或 Flexible 元件。

### 內外邊距

若想同時控制元件的內外邊距、長寬、背景色等等，可以使用 Container 元件。Container 還可以設定邊框、圓角等等，是相當常用的元件之一。例如：

```dart
var comp = Container(
  margin: const EdgeInsets.symmetric(horizontal: 16.0),
  padding: const EdgeInsets.all(16.0),
  color: Colors.grey.shade200,
  height: 100,
  width: 100,
  child: const Text('我是文字'),
);
```

![image-20230412222104812](container.png)

## 元件排版

### 統一內縮

想要一個區塊內的元件都有同樣的內縮並且有背景色，那我們可以使用 `Container` 搭配 `Column` 這個元件來完成。

```dart
var comp = Container(
  padding: const EdgeInsets.all(16),
  color: Colors.grey.shade200,
  child: Column(
    children: [
      const Text('文字1'),
      const Padding(padding: EdgeInsets.only(top: 16.0)),
      const Text('文字2'),
    ],
  ),
);
```

![column](column.png)

### 併排呈現

當有好幾個元件，想要再同一排中做呈現的時候，可以使用 `Row` 這個元件。

```dart
var comp = Container(
  padding: const EdgeInsets.all(16),
  color: Colors.grey.shade200,
  child: Row(
    mainAxisAlignment: MainAxisAlignment.start,
    children: [
      const Text('文字1'),
      const Padding(padding: EdgeInsets.only(left: 16.0)),
      const Text('文字2'),
    ],
  ),
);
```

![row](row.png)

## 進階使用

### 自動寬高度

如果希望併排的元件中，有一些是固定寬度，而剩下的元件則佔滿剩餘的全部寬度，可以使用 Expanded。在 Row 中使用 Expanded 元件時，會讓剩下的空間平均分配給 Expanded 元件，因此可以讓剩下的元件都自動調整寬度。

> 在 `Column` 中也可以使用 `Expanded` 元件來達成自動調整高度的效果

```dart
var comp = Container(
  padding: const EdgeInsets.all(16),
  color: Colors.grey.shade200,
  child: Row(
    mainAxisAlignment: MainAxisAlignment.start,
    children: [
      const Expanded(
              child: Text('文字1'),
      ),
      const Padding(padding: EdgeInsets.only(left: 16.0)),
      const Text('文字2'),
    ],
  ),
);
```

![expand-in-row](expand-in-row.png)

### Flex

如果想要更精細地控制元件的寬度和高度，可以使用 Flexible 元件。設定 flex 屬性，讓元件按照比例分配空間。

```dart
var comp = Row(
  mainAxisAlignment: MainAxisAlignment.start,
  children: [
    // 只會佔用剩下的空間
    Flexible(
      child: Container(
        color: Colors.amber,
        width: 100,
      ),
    ),
    Flexible(
      // 佔用 10 等份
      flex: 10,
      child: Container(
        color: Colors.teal,
      ),
    ),
    // 固定大小
    Container(
      color: Colors.cyan,
      width: 200,
    ),
  ],
);
```

看到底下結果是第一個元件並不會真正佔用到 100，反而會是第二和第三個元件所佔用的空間後剩下來的才會分配給他。

![flex-in-row](flex-in-row.png)

## 有雷要注意

### 併排呈現

當使用 `Row` 的時候，底下的元件總寬度必須要在裝置的寬度之內，不然就會直接噴錯，在這邊是建議直接加上 `Flexiable` 讓元件平均分配，避免這個問題產生。

```dart
var comp = Row(
  mainAxisAlignment: MainAxisAlignment.start,
  children: [
    Container(
      color: Colors.amber,
      width: 200,
    ),
    Container(
      color: Colors.teal,
      width: 200,
    ),
    Container(
      color: Colors.cyan,
      width: 200,
    ),
  ],
);
```

![not-enough-space-in-row](not-enough-space-in-row.png)

![error-message-for-row](error-message-for-row.png)

### 垂直呈現

在 Flutter 中，預設的排版方式是垂直往下，當元件超過裝置的高度時，也是一樣會噴錯，那這時候可以加上捲軸的元件 `SingleChildScrollView`，就可以解決高度不夠的問題。

```dart
var comp = SingleChildScrollView(
  child: Column(
    children: [
      Container(
        color: Colors.amber,
        height: 500,
      ),
      Container(
        color: Colors.teal,
        height: 200,
      ),
      Container(
        color: Colors.cyan,
        height: 300,
      ),
    ],
  ),
);
```

![over-height](over-height.png)

## 結論

在 Flutter 的世界中，當你的排版比較複雜，要往下排又有元件要併排，然後還要控制內外邊距時，就會有很多階層的產生，這時候會建議適度的抽成元件，讓主要的 build 比較容易懂。

如果你跟我一樣主要是網頁的開發者，記得請拋棄你的既定印象和習慣，多多使用元件來搭配，才能排版出符合設計的樣式。

```dart
SingleChildScrollView(
  child: Container(
    padding: const EdgeInsets.all(38),
    color: const Color.fromRGBO(217, 217, 217, 0.5),
    child: Column(
      children: [
        _LastNameInput(),
        const Padding(padding: EdgeInsets.only(top: 12.0)),
        _FirstNameInput(),
        const Padding(padding: EdgeInsets.only(top: 12.0)),
        _BirthdayInput(),
        const Padding(padding: EdgeInsets.only(top: 12.0)),
        _GenderInput(),
        const Padding(padding: EdgeInsets.only(top: 12.0)),
        _PhoneInput(),
        const Padding(padding: EdgeInsets.only(top: 12.0)),
        _EmailInput(),
        const Padding(padding: EdgeInsets.only(top: 12.0)),
        _PasswordInput(),
        const Padding(padding: EdgeInsets.only(top: 12.0)),
        _ConfirmedPasswordInput(),
        const Padding(padding: EdgeInsets.only(top: 12.0)),
        _ReceiveNewsInput(),
        const Padding(padding: EdgeInsets.only(top: 12.0)),
        _AgreePolicyInput(),
        const Padding(padding: EdgeInsets.only(top: 40.0)),
        _RegisterButton(),
      ],
    ),
  ),
);
```
