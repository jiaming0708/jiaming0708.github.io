---
title: 讓vscode讀懂alias path
date: 2020-01-31 17:00:14
categories:
- Front-end
- vscode
tags:
- vscode
- webpack
---

在專案開發的時候，為了讓import的路徑短一點，通常都會用alias來處理，例如`../../../components/Button`會用`@components/Button`來做為取代

> 這邊不討論用auto import還是用alias的好壞

<!-- more -->

# code

```javascript
// import Button from '../../../components/Button';
// alias
import Button from '@components/Button';
```

# webpack

首先我們會在webpack加上resolve的設定

> 相關資料可以參考[官網](https://webpack.js.org/configuration/resolve/)

```javascript
{
	resolve: {
		alias: {
			'@components': path.resolve(__dirname, 'src/components')
		}
	}
}
```

這樣的設定是為了讓編譯看的懂，但是vscode還是看不懂，當你滑鼠移動到 `@components/Button` 就無法知道真正的路徑，也沒辦法快速的用*f12*移動到檔案中看內容

# jsconfig

為了要能夠讓vscode讀懂alias，可以在根目錄底下建立`jsconfig.json`，在裡面加上下面的設定，儲存後不用重開只要等個幾秒，就可以測試是不是有效了！

> 相關資料可以參考[官網](https://code.visualstudio.com/docs/languages/jsconfig)

```json
{
  "compilerOptions": {
    "target": "es2017",
    "baseUrl": "./",
    "paths": {
      "@components/*": ["src/components/*"]
    }
  },
  "exclude": ["node_modules"]
}
```

在我的專案中，如果只有設定`baseUrl`和`paths`是沒有效果的，必須要搭配上`target`才能正確的解析，在使用上需要注意一下，避免設定完了結果還是不會work