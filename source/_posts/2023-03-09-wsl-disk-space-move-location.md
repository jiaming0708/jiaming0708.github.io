---
title: WSL空間擴增問題排除
date: 2023-03-09 20:47:23
updated: 2023-05-11 11:31:23
categories:
- Windows
tags:
- Windows
- Docker
thumbnail:
---

最近發現工作電腦的c槽突然間快滿了，查了一下被哪些東西吃掉，才發現一個噁心的怪獸，就是 WSL 的虛擬硬碟 vhdx 吃掉了 30G，底下會說明該怎麼解決這個問題。

<!-- more -->

在 Windows 安裝 WSL 大多是為了使用 Docker Desktop 這個軟體，WSL 的虛擬硬碟檔案則是放在使用者資料夾下的 `AppData\Local\Docker\wsl\data`，可以看到一個叫做 `ext4.vhdx` 的檔案。
有另外安裝 ubuntu 之類的系統的話，檔案路徑則不一樣的是在 Packges 並且根據你安裝的系統後面加上一些獨立的名稱， `AppData\Local\Packages\CanonicalGroupLimited.Ubuntu20.04onWindows_79rhkp1fndgsc\LocalState` 。

當我們使用完畢 container 關閉後，vhdx 的檔案並不會因此而釋放掉所使用到的空間，每次使用就會吃掉一點，不斷的擴增並且是沒有限制！！
這個問題從 2019 年就有人開 [issue](https://github.com/microsoft/WSL/issues/4699)，但都還沒有辦法解決，裡面有些人提出一些作法是把檔案路徑搬走，至少不要讓系統c槽的空間被吃光。

底下是根據裡面討論到的有效解法，蠻建議安裝完後就移動這個位置，讓自己的 windows 不會因此有空間不足而產生的效能問題。

> 記得要使用系統管理員的權限執行，不然最後連結檔案的步驟會失敗

```powershell
# 其他槽的位置
$newLocation = "E:\WSL2\"

cd "~\AppData\Local\Docker\wsl\data"
# cd "~\AppData\Local\Packages\{distro folder name}\LocalState"
wsl --shutdown
mkdir $newLocation -Force
mv ext4.vhdx $newLocation
cd ..
rm "data"
# 若是 distro 的話是放在 LocalState 目錄下
# rm "LocalState"

# 使用連結檔案的方式，讓實體是在其他指定的位置
New-Item -ItemType SymbolicLink -Path "data" -Target $newLocation
# New-Item -ItemType SymbolicLink -Path "LocalState" -Target $newLocation
```

> 2023/04/07 補充

~~另外補充給使用 Podman 的朋友，路徑跟以上完全不同，檔案是在 `.local` 資料夾中， `~\.local\share\containers\podman\machine\wsl`
podman 可以建立多個，預設叫做 `podman-machine-default` ，若有改名字記得底下的內容要跟著調整~~

> 抱歉，隔天重開機後，machine 會一直說 permission denied，還需要再研究一下

```shell
# 其他槽的位置
$newLocation = "E:\WSL2\"

cd "~\.local\share\containers\podman\machine\wsl\wsldist\podman-machine-default"
# 關閉 podman machine
podman machine stop podman-machine-default

mkdir $newLocation -Force
mv ext4.vhdx $newLocation
cd ..
rm "podman-machine-default"

# 使用連結檔案的方式，讓實體是在其他指定的位置
New-Item -ItemType SymbolicLink -Path "podman-machine-default" -Target $newLocation
# 啟動 podman machine
podman machine start podman-machine-default
```

> 2023/05/11 再次補充

使用 Podman 如果要改路徑的話，不能用上面的方法，要從根本解決也就是一開始安裝的時候，就直接讓目錄放在預期的目錄下。
如同前面預設放在 `.local` 下，可以透過修改環境變數來變更目錄，打開 terminal 執行以下的指令。

```shell
setx XDG_DATA_HOME "E:\WSL2"
```

如果已經有安裝的 podman machine，就記得先移除再重新安裝。

```shell
podman machine rm podman-machine-default
podman machine init
podman machine start
```

接著到前面設定的路徑確認，可以看到底下會有一個 `containers` 的資料夾。
這樣就做到修改 podman 的 ext4 檔案搬移到其他槽的目的。

> 2023/09/11 補充說明

前面的作法在 `Docker Desktop v4.22.1` 及 `WSL v1.2.5.0` 環境下，已經無效，標準的作法可以參考這篇 gist

{% gist 1d58d40f217dfb21f0aa54517583f8d3 README.md  %}
