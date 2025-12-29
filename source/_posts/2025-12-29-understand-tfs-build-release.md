---
title: 搞懂 TFS Build/Release
date: 2025-12-29 08:48:35
updated: 2025-12-29 08:48:35
categories:
- DevOps
tags:
- DevOps
- TFS
thumbnail:
---

TFS 全名是 Team Foundation Server，可以說是 Azure DevOps Server 的前身，這套是 2005 年上市的距今已經 20 年。
在進現在的公司之前，我只有用過 Azure DevOps 耳聞過 TFS，進來使用後才發現這個系統有些觀念和目前常見的 CI/CD 平台(例如 Github Action)是不太一樣的。
問了同事也說的不是很清楚，這邊做個筆記。

<!-- more -->

## 搞懂 Build 和 Release

一開始在 TFS 中找不到 CI/CD 的字樣，而是看到 Build 和 Release，在設定裡面看到 Build 有分 `正式`、`測試`、`QC`，我心裡想著裡面有什麼差異...
沒有搞懂之前，很難真的好好使用這個平台工具，更不用說去優化或是改善流程。

### Release

從 Release 開始看起

![new-release](new-release.png)

第一步是要先選擇 artifact (build 後的產出)

![select-artifact-source](select-artifact-source.png)

接著是一個我個人偏向奇怪的設定，就是設定完 artifact 還要額外點啟用，不然根本不會觸發...
> 雖然知道這功能是為了可以讓設定還在只是先暫時關閉，但預設沒有啟用對於初次使用的人來說就很困擾

![artifact-trigger](artifact-trigger.png)

![artifact-trigger-setting](artifact-trigger-setting.png)

第二步開始就是可以部署到各個環境上，裡面的動作就看公司環境來決定要怎麼做
以我們來說就會有
- 下分流 (等待 n 秒)
- 部署新服務
- 確認服務運行
- 上分流

在這邊可以分不同的階段，可以對應不同的行為

![release-phase](release-phase.png)

以上面的步驟來說，我們是這樣區分

- Agent phase
  - 下分流 (等待 n 秒)
- Deploy Group phase
  - 部署新服務
  - 確認服務運行
- Agent phase
  - 上分流

Agent 和 Deploy Group 最主要的差異是在哪台機器上運行，Agent 通常就是跟著 TFS 主機或是另外其他幾台主機，通常都是專屬服務
Deploy Group 的話則是運行在目標主機，也就是部署的主機上，對我公司來說就是 iis 或是 ec2

### Build

既然前面的 Release 只處理部署的部分，那接著就是來設定 Build

首先設定程式碼的來源，以及要用哪一個 Agent，在 TFS 可以起很多個 Agent 給不同目的使用，對比 Github Action 也就是 Runner
接著就是設定要做些什麼樣的動作在這個階段，以 dotnet 來說，就可以設定三個步驟 restore, build, test

![build-task](build-task.png)

在 Build 有一個設定叫做 Trigger，當某個分支有新 commit 進來的時候觸發

![build-trigger](build-trigger.png)

### 小結

Release 是以 artifact 為觸發點，後續再來根據需求來部署到不同的環境。對比 Github Action 的話，那這個是通常會是 CD 流程中的最後一步驟。

Build 的話則要看有沒有設定 Trigger，當有設定 Trigger 就是屬於 CD 流程的前半段；若沒有的話，則算是 CI 的流程；但更精確的說法應該是只要在 Build 有設定產出 artifact 就可以當作 CD 流程的前半段。

## 實際情境

環境有正式、測試，並且需要一個 CI 的情境下，我們這樣建立 Build 和 Release

### Build

建立三個 Build，兩個給 CD 一個給 CI
- 正式，設定 trigger 為 master
- 測試，設定 trigger 為 develop
- CI，不設定 trigger

建立兩個 Release，artifact 來源以及目標主機都不同
- 正式
- 測試

看起來好像設定完了，實際上還少了一塊，CI 由誰觸發
在 TFS CI 的觸發設定是完全分開來的，都不在前面的區塊內，而是在 code -> branch -> branch policy 的設定中

![branch-policy](branch-policy.png)

在 `Build Validation` 那個區塊，要加上剛剛所設定的 CI 並且啟用，就會在有建立 PR 到這個分支時觸發

## TFS vs GitHub Actions 差異對比

> 本段由 AI 整理上述後產出

從上面的使用經驗，可以整理出 TFS 和 GitHub Actions 在設計理念和使用上的一些關鍵差異。

### 核心概念差異

| 面向 | TFS | GitHub Actions |
|------|-----|----------------|
| **流程架構** | Build + Release 分離設計 | 單一 Workflow 統一管理 |
| **觸發來源** | Build: 程式碼<br>Release: Artifact | 統一由程式碼事件觸發 |
| **設定方式** | UI 介面點選 | YAML 檔案定義 |
| **設定儲存** | TFS Server 資料庫 | `.github/workflows/` 目錄 |
| **版本控制** | 設定與程式碼分離 | 設定與程式碼一起版控 |

### 觸發機制的差異

**TFS 的多層觸發設計：**

1. **PR 觸發 CI**：在 Branch Policy 設定
2. **Push 觸發 Build**：在 Build 的 Trigger 設定
3. **Artifact 觸發 Release**：在 Release 的 Artifact 設定（還要記得啟用！）

```
程式碼事件 → Build → Artifact → Release → 部署
    ↑           ↑         ↑
  Branch     Build    需手動啟用
  Policy    Trigger    的觸發器
```

**GitHub Actions 的統一觸發：**

所有觸發條件都在 workflow 檔案的 `on` 區塊定義：

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

### 實際使用情境對比

#### 情境一：PR 時執行測試（CI）

**TFS 做法：**
1. 建立一個 Build（不設定 Trigger）
2. 到 Branch Policy 設定 Build Validation
3. 選擇剛建立的 Build
4. 勾選啟用

**GitHub Actions 做法：**
```yaml
# .github/workflows/ci.yml
on:
  pull_request:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: dotnet test
```

#### 情境二：推送到 develop 自動部署測試環境（CD）

**TFS 做法：**
1. 建立 Build，設定 Trigger 為 develop 分支
2. Build 設定產出 artifact
3. 建立 Release，選擇該 Build 的 artifact
4. Release 設定部署步驟
5. **記得啟用** Release 的 Artifact Trigger

**GitHub Actions 做法：**
```yaml
# .github/workflows/deploy-dev.yml
on:
  push:
    branches: [develop]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: dotnet build
      - run: dotnet publish
      - name: Deploy to Test
        run: # 部署指令
```

### 優缺點分析

#### TFS 的特點

**優點：**
- UI 操作對非技術人員友善
- Build 和 Release 職責分明，適合大型企業的權限管控
- Release 可以重複使用同一個 artifact 部署到不同環境
- Agent 和 Deploy Group 的區分讓執行位置更明確

**缺點：**
- 設定分散在多處（Build Trigger、Branch Policy、Release Trigger）
- Release Trigger 預設關閉容易遺漏
- 設定無法版本控制，難以追蹤變更歷史
- 要設定完整的 CI/CD 需要建立多個 Build

#### GitHub Actions 的特點

**優點：**
- 所有設定在一個 YAML 檔案，一目了然
- 設定與程式碼一起版控，方便追蹤和回溯
- 社群有大量現成的 actions 可以使用
- 容易複製和分享設定

**缺點：**
- 需要學習 YAML 語法
- 每次部署都會重新建置（除非使用 artifact）
- 權限管控較不直覺

### 設計哲學的不同

**TFS：瀑布式的階段思維**
- Build 負責建置和測試
- Release 負責部署
- 兩者透過 Artifact 串接
- 適合傳統企業的變更管理流程

**GitHub Actions：持續整合的流式思維**
- 從程式碼到部署是一個連續的流程
- 所有步驟定義在同一處
- 更符合現代 DevOps 的 Infrastructure as Code 理念

## 結論

從上面的一些可以發現，TFS 的設計思維跟現在有一定的落差，再加上當時沒有強調 `xxx as Code` 的概念，因此都是透過 UI 的設定並且將設定保留在 server 端，嘗試深入的理解平台工具，可以幫助我們更容易的去使用，也才有機會去調整沒那麼適合現代的流程。
