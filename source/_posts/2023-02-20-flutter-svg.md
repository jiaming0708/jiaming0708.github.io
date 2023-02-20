---
title: '[Flutter] SVG應用'
categories:
  - Flutter
tags:
  - Flutter
date: 2023-02-20 22:13:56
thumbnail: /images/flutter.png
---

在網頁使用 icon 的時候，會想要採用 svg，主要原因是可以渲染並且不會失真，而且可以做動態渲染的效果，接下來看看如何在 flutter 使用 svg 來開發。

<!-- more -->

## 安裝使用

執行以下的指令安裝套件 [flutter_svg](https://pub.dev/packages/flutter_svg)

```shell
flutter pub add flutter_svg
```

在 `pubspec.yml` 可以看到增加這行（若沒有請自己加上）。

```yaml
dependencies:
  flutter_svg: ^2.0.0+1
```

安裝後，在程式裡面引用並且使用 `SvgPicture` 這個類別。

```dart
import 'package:flutter_svg/flutter_svg.dart';
```

## 檔案在本地

當 icon 檔案在本地的時候，要使用的是 `SvgPicture.asset` 這個方法。

```dart
var icon = SvgPicture.asset('assets/add.svg');
```

直接放路徑是最基本的用法，若是希望指定元件的大小，則可以加上 `width` 及 `height` 兩個參數。

```dart
var icon = SvgPicture.asset('assets/add.svg', width: 16, height: 16);
```

## 檔案在網路

使用上跟前面差不多，主要是方法改成 `network`。

```dart
var icon = SvgPicture.network(path);
```

要指定大小的話，一樣可以加上 `width` 及 `height` 兩個參數。

## 改變顏色

把 add 改成紅色，只需要加上 `color` 這個參數。

```dart
final Widget svgIcon = SvgPicture.asset(
  'assets/add.svg',
  width: 16,
  height: 16,
  color: const Color(0xFFFF0000),
);
```

## 內縮效果

如果 icon 是需要有 padding 的話，就可以搭配 `Container` 這個物件來完成。

```dart
var icon = SvgPicture.asset('assets/add.svg');
// icon 最終的寬高為 16
var iconWithPadding = Container(
  margin: const EdgeInsets.symmetric(horizontal: 8.0),
  height: 32,
  width: 16,
  child: icon,
);
```

> 在 flutter 的世界中，效果都是透過疊加的方式做呈現，很少有一個元件就能達成全部的效果。
> 如果你跟我一樣平常主要接觸是網頁開發，這個地方要稍微習慣一下。
