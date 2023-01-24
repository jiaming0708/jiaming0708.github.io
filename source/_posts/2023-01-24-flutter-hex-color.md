---
title: '[flutter] 使用 16 進制定義顏色'
date: 2023-01-24 14:16:00
categories:
  - Flutter
tags:
  - Flutter
thumbnail: /images/flutter.png
---

在 Flutter 中有提供好幾種方法可以來定義 `Color` 這個物件，這篇會介紹如何使用這些方法。

<!-- more -->

## 已知的顏色

- 使用到的是常用的顏色，例如黑、白、紅、綠、黃等等的，可以直接使用 `Colors` 來找到需要的顏色。

```dart
Color danger = Colors.red;
```

- 若是要使用 0-255 間的數字來設定顏色，則有兩種作法，主要的差異在於 `opacity`

```dart
// opacity 於最後一個參數，值介於 0-1 的浮點數
Color danger = Color.fromRGBO(255, 0, 0, 1);
// opacity 於第一個參數，值介於 0-255 的整數
Color danger = Color.fromARGB(255, 255, 0, 0);
```

- 使用 hex 定義，組成方式為 alpha red green blue

```dart
Color danger = Color(0xFFFF0000);
```

## 未知的顏色

當顏色的不是先定義好，而是由 API 的資料來決定的話，會需要將字串做個轉換；在這個情況下，建議跟後端溝通拿到的顏色字串為 hex 格式，也就是 `RRGGBB`。
在根據前面所說的 hex 顏色定義方法，適度的補上 alpha 值，在透過 int 的方式來宣告成 `Color` 物件。

```dart
Color hexToColor(String hexString) {
  final buffer = StringBuffer();
  if (hexString.length == 6 || hexString.length == 7) buffer.write('ff');
  buffer.write(hexString.replaceFirst('#', ''));
  return Color(int.parse(buffer.toString(), radix: 16));
}
```

可以宣告成一個新的物件，或是擴充功能，就看怎麼用比較順手了。

```dart
extension ColorExtension on String {
	Color toColor(String hexString) {
    //...
	}
}
Color danger = '#ff0000'.toColor();
```

