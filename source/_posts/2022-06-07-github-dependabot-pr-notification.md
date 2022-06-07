---
title: 使用 Github Action 檢查套件更新並發送 PR
date: 2022-06-07 20:14:13
categories:
- Github Action
tags:
- Github Action
- Telegram
---

放在 Github 上面的 Repository 可以開啟相依套件的更新檢查，甚至會自動幫你發送 PR，今天來看一下怎麼樣設定並在最後發出通知。

<!-- more -->

# 開啟 Dependabot

> Github 的設定可能會改變位置，這是到目前為止的設定，如果找不到，請試著用類似的字眼去找。

到你的 Repository 中，移到 `Security` 頁籤，將 `Dependabot alerts` 啟用。

![repo-security](repo-security.png)

![setting-code-security](setting-code-security.png)

系統會帶你到 Setting 頁籤，先 `Dependabot security updates` 啟用，就可以看到第一個項目 `Alert` 也會被啟用。

![setting-enabled-security-update](setting-enabled-security-update.png)

接著再將 `Dependabot version updates` 啟用，系統會帶你去建議一個 yaml 檔案。

![create-dependabot-yaml](create-dependabot-yaml.png)

這邊要注意的是， `package-ecosystem` 是留空要自己填寫，可以在 [Configuration options for the dependabot.yml file - GitHub Docs](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#package-ecosystem) 中找到自己的套件管理，像我的是用 yarn 做管理。

做完以上的設定以後，只要 Github 檢查到相依套件中有任何危險或是更新，就會建立一個 PR。

# 當 PR 建立時通知

當 Dependabot 建立 PR 的時候，除了上去 Github 看以外，就是要透過收信的方式，但我個人是不喜歡信件的通知，比較喜歡用通訊軟體即時的收到通知，在這時候我們就可以使用 Github Action 來做到這件事情。

剛剛啟用 `Dependabot version updates` 所建立的 `dependabot.yml` 是放在 `.github` 這個目錄底下，Github Action 也是一樣在這目錄底下，新增一個目錄 `workflows`，建立一個叫做 `notify-pull-request.yml` 的檔案，並將底下內容貼上。

```yaml
name: Send Message on Pull Request open.

on:
  pull_request:
    types: [opened, reopened]

jobs:
  sendMessage:
    runs-on: ubuntu-latest

    steps:
      - uses: hmarr/debug-action@v2
      - name: send telegram message
        uses: appleboy/telegram-action@master
        with:
          to: ${{ secrets.TELEGRAM_CHAT_ID }}
          token: ${{ secrets.TELEGRAM_TOKEN }}
          # 注意 tg message 長度有限制，不要放太多內容會爆掉
          message: |
            Author: ${{ github.event.pull_request.user.login }}
            Title: ${{ github.event.pull_request.title }}
            Link: ${{ github.event.pull_request.html_url }}
```
> 關於 Github Action 的基礎介紹，可以參考 {% post_link github-action-and-package %}，或是 [不僅是程式碼代管平台 - Github 能做些什麼? :: 2021 iThome 鐵人賽](https://ithelp.ithome.com.tw/users/20091494/ironman/4464)

根據[文件](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request) PR 預設行為是 `opened`, `synchronize`, or `reopened`，這邊重新設定將 `synchronize` 去掉。

> Telegram 的設定教學可以參考 [[GitHub Actions\] 發送 Telegram 訊息 | 全端開發人員天梯 (fullstackladder.dev)](https://fullstackladder.dev/blog/2021/11/01/github-actions-send-telegram/)。

接著我們要來設定 secrets，現在的設定有分為 `Actions` 及 `Dependabot` 兩個，雖然發送 PR 的通知是透過 Action，但建立 PR 的是 bot，因此兩個地方都必須要將 Telegram 的設定放進去。

![setting-secret-actions](setting-secret-actions.png)

![setting-secret-dependabot](setting-secret-dependabot.png)

都設定好以後，不管是誰建立的 PR，我們都可以在 Telegram 收到啦!!

# Reference

- [Configuration options for the dependabot.yml file - GitHub Docs](https://docs.github.com/en/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file#package-ecosystem)
- [Events that trigger workflows - GitHub Docs](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request)
- [[GitHub Actions\] 發送 Telegram 訊息 | 全端開發人員天梯 (fullstackladder.dev)](https://fullstackladder.dev/blog/2021/11/01/github-actions-send-telegram/)
- [不僅是程式碼代管平台 - Github 能做些什麼? :: 2021 iThome 鐵人賽](https://ithelp.ithome.com.tw/users/20091494/ironman/4464)

