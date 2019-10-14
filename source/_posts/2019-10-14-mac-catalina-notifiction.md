---
title: '[mac] Catalina的notification設定'
date: 2019-10-14 15:36:11
categories:
- mac
tags:
- mac
---

上禮拜官方release了最新的作業系統，當然要跟上體驗最新的Catalina這個作業系統，但是蘋果提昇了Catalina的一些安全性管理，如果你跟我一樣討厭被notification打擾，那可以看一下我怎麼設定

<!-- more -->

在更新完系統後，開啟各種app就會開始提示你是否允許notification，如果你跟我一樣都按 `Not allow(不允許)`然後就GG惹，會發現app的所有通知都不見（包含badge）

# 開啟notification

接著我們要來重新設定一下

1. 在 `System Preferences` 找到 `Notifications` 並且打開

![System Preferences](system_preferences.png)

2. 找到不被允許的app，並且設定成允許通知

![Notifications](notifications.png)

3. 對我來說只需要有badge就可以，其他都不需要，將設定改成只剩下badge就好

![Enable Notification](enable_notification.png)

# slack設定

只要在Notifications中沒有被列出來的app就代表沒有需要通知的功能，如果你的slack是沒有被列在裡面的，可以參考一下我的作法

前面有說到我很不喜歡太多的notification，因此我把slack設定成都沒有任何提醒只會顯示badge

![Slack preferences](slack_preferences.png)

在這個情況下，對於Catalina來說是等於沒有需要通知這個功能，因此完全不會被列在 `Notifications` 中，那要做的事情很簡單

![slack notification](slack_notification.png)

1. 將 `Nothing` 改成其他的任何一種
2. 新的訊息進來時候系統就會問你是否要開啟或是關閉notification的功能
3. 一樣到 `Notifications` 做詳細的設定
4. 看你的習慣要不要再把slack的設定改回去成 `Nothing`

# 結論

照習慣還是要結論一下，因為新系統的安全性提高，因此有些地方要重新設定，以免跟之前的使用習慣都不一樣。

>  如果你還沒升級或是升級以後遇到類似的情況，希望這篇能對你有所幫助