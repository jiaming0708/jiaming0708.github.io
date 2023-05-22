---
title: WSL 進階設定
date: 2023-05-19 13:14:29
updated: 2023-05-19 13:14:29
categories:
- Windows
- WSL
tags:
- Windows
- WSL
thumbnail:
---

WSL 2 預設的記憶體在 windows 10 build 20175 版本之前能夠使用到 80%，但在之後的版本最多就是 50% 或是 8GB，對於現在電腦動輒 32G 的情況來說，這樣的使用量實在太少，這篇來分享一下怎麼做調整。

<!-- more -->

## .wslconfig

WSL 2 的設定檔名叫做 `.wslconfig`，存放的目錄在 `%UserProfile%` 下，若沒有這個檔案的話先行建立。

> 電腦配備
> CPU: 16 cores, 24 logical processors
> MEM: 32G

以下是我調整的內容，主要是要讓記憶體能夠使用多一點，並且把 swap 的檔案調整到非系統槽。

```bash
# Settings apply across all Linux distros running on WSL 2
[wsl2]

# Limits VM memory to use no more than 4 GB, this can be set as whole numbers using GB or MB
memory=20GB 

# Sets the VM to use two virtual processors
processors=12

# Sets amount of swap storage space to 8GB, default is 25% of available RAM
swap=8GB

# Sets swapfile path location, default is %USERPROFILE%\AppData\Local\Temp\swap.vhdx
swapfile=D:\\WSL\\wsl-swap.vhdx
```
