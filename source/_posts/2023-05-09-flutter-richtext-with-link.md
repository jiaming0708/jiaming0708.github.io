---
title: '[Flutter] 實現帶有連結文字的 RichText'
date: 2023-05-09 21:00:00
updated: 2023-05-09 21:00:00
categories:
  - Flutter
tags:
  - Flutter
thumbnail: /images/flutter.png
---

在註冊時通常會服務營運端會放置一些規章要求你同意，其中一種的表現方式是在一段的文字中帶有連結，讓你點擊過去，或是直接勾選同意。接下來要展示如何在 Flutter 中實現這樣的效果。

<!-- more -->

其實，在前一篇 {% post_link flutter-custom-font-family %} 最後有提到這次的元件。沒錯！就是使用 `Text.rich` 或是 `RichText` 在搭配上 `TextSpan`。

底下我們就來展示一段文字中有些文字是可以點擊連結開啟網頁，在開始寫之前，請先安裝 `url_launcher` 套件，以便開啟網頁。

> url_launcher 版本 6.1.7

```shell
flutter pub add url_launcher
```

為了方便，直接使用 [Discord 的註冊](https://discord.com/register) 頁面中的文字，以下是程式碼

> 註冊即代表同意 Discord 的 [服務條款](https://discord.com/terms) 及 [隱私權政策](https://discord.com/privacy)。

```dart
RichText(
  text: TextSpan(
    children: <TextSpan>[
      const TextSpan(text: '註冊即代表同意 Discord 的 '),
      TextSpan(
        text: '服務條款',
        style: const TextStyle(
          color: Colors.blue,
        ),
        recognizer: TapGestureRecognizer()
          ..onTap = () {
            launchUrl(Uri.parse('https://discord.com/terms'));
          },
      ),
      const TextSpan(text: ' 及 '),
      TextSpan(
        text: '隱私權政策',
        style: const TextStyle(
          color: Colors.blue,
        ),
        recognizer: TapGestureRecognizer()
          ..onTap = () {
            launchUrl(Uri.parse('https://discord.com/privacy'));
          },
      ),
      const TextSpan(text: '。')
    ],
    style: const TextStyle(
      color: Colors.black,
    ),
  ),
)
```

從上面的範例中，預設的文字顏色是黑色，有連結的文字是藍色，讓使用者更容易的辨認。在 `TextSpan` 中透過 `recognizer` 的屬性，使用 `TapGestureRecognizer` 來處理單次點擊的行為，裡面直接開啟外部的網頁。
