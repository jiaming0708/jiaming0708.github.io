---
title: 專案設定 - vscode 以及 nvmrc
date: 2020-07-19 23:26:00
categories:
- Frontend
- Project
tags:
- vscode
- nvm
---

open source總是有些不一樣的東西可以挖，今天為了要找一些使用action的範例，反而讓我找到了一些不一樣的設定，可以幫助我們。

<!-- more -->

# .vscode

當改變了一些設定是只有為了專案的時候，vscode 就會產生一個 `settings.json` 的檔案在根目錄底下，除了設定以外，vscode 也支援 推薦 extensions 給專案的使用者，只要在 `.vscode` 的目錄下建立 `extensions.json` 即可，範例如下：

```json
{
  "recommendations": [
    // format: {publisher}.{extension}
    "ms-vscode.vscode-typescript-tslint-plugin",
    "dbaeumer.vscode-eslint",
    "msjsdiag.debugger-for-chrome"
  ]
}
```

若是有一些推薦的設定，但不希望強制的話，就可以在 `.vscode` 的目錄下，建立一些 `recommended-*` 的檔案，由使用者自行改檔名套用，例如： `.vscode/recommended-settings.json` 可以複製成 `.vscode/settings.json`

# nvmrc

手邊每一個 node 的專案都不同版本，那為了確保版本的正確性，可以使用 `.nvmrc` 的檔案搭配 zsh script 自動載入版本，這樣就可以避免每次開啟 terminal 都還要手動切換版本的問題。

首先要在專案中產生 `.nvmrc` 的檔案

```shell
node -v > .nvmrc
```

接著到 `.zshrc` 中將以下的 script 貼上

```shell
# place this after nvm initialization!
autoload -U add-zsh-hook
load-nvmrc() {
  local node_version="$(nvm version)"
  local nvmrc_path="$(nvm_find_nvmrc)"

  if [ -n "$nvmrc_path" ]; then
    local nvmrc_node_version=$(nvm version "$(cat "${nvmrc_path}")")

    if [ "$nvmrc_node_version" = "N/A" ]; then
      nvm install
    elif [ "$nvmrc_node_version" != "$node_version" ]; then
      nvm use
    fi
  elif [ "$node_version" != "$(nvm version default)" ]; then
    echo "Reverting to nvm default version"
    nvm use default
  fi
}
add-zsh-hook chpwd load-nvmrc
load-nvmrc
```

接著用一個新視窗來開啟專案，就可以看到 zsh 自動載入 node 版本

```shell
> cd /Grapefruit
Found '/Grapefruit/.nvmrc' with version <v12.6.0>
Now using node v12.6.0 (npm v6.9.0)
```

> 實測以後，發現這個功能雖然好用，但開啟的速度就會變比較慢一點點，如果在意的話，建議不要使用

# 參考資料

* [angular vscode readme](https://github.com/angular/angular/blob/master/.vscode/README.md)
* [vscode recommend extensions](https://code.visualstudio.com/docs/editor/extension-gallery#_workspace-recommended-extensions)
* [nvmsh](https://github.com/nvm-sh/nvm#zsh)