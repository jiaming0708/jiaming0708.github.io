---
title: Jenkins 與 Gitea 整合
date: 2022-03-15 15:04:59
updated: 2022-03-15 15:04:59
categories:
- CI/CD
tags:
- CI/CD
- Jenkins
- Gitea
---

Dotnet framework 的 docker image 只有 window 版本，要特別將 Docker Desktop 切換成 Windows container 才能跑，為了讓 Dotnet framework 及 Dotnet core 能同時跑在一個 CI/CD 的平台，最後選擇了 Jenkins 這位老爺爺。

> 必須說老爺爺的功能很強大，但也真的很不友善...很多東西一直查資料，只好寫一篇來筆記一下orz

<!-- more -->

# 安裝 Jenkins

Jenkins 在 Windows 的安裝方法有兩種，第一個是透過 [jenkins/jenkins - Docker Image | Docker Hub](https://hub.docker.com/r/jenkins/jenkins)，另一個是走標準安裝的方法，我選擇後者標準安裝的作法。

> 不使用 docker 是因為要 volumn 太多東西，反而造成自己困擾
> 但很建議要先體驗的朋友，先透過 docker

> 環境資訊
> Jenkins 2.332.1 LTS
> Windows 11

## 設定使用者

選完安裝目錄後，要指定一個使用者作為服務執行用，這邊我採用的是目前使用者。

![jenkins-install-logon](./jenkins-install-logon.png)

輸入後得到一個錯誤訊息 **This account either does not have the privilege to logon as a service or the account was unable to be verified**，這表示帳號登入成功但沒有權限作為登入服務；官方的介紹在這 [Windows (jenkins.io)](https://www.jenkins.io/doc/book/installing/windows/#invalid-service-logon-credentials)。

![invalid-logon](./invalid-logon.png)

接著要將登入的使用者加上可以登入服務的權限，打開 `本機安全性原則(Security Configuration Management)`，開啟 `本機原則` > `使用者權限指派` > `以服務方式登入`。

![security-configuration-management](./security-configuration-management.png)

找到使用者後加入。

![search-user](./search-user.png)

再次檢查使用者，可以往下一步了!

![jenkins-install-logon-validate](./jenkins-install-logon-validate.png)

## 安裝 JAVA

Jenkins 需要 JAVA 的執行環境，安裝過程中會要求指定目錄，可以直接到[官網下載](https://java.com/zh-TW/download/ie_manual.jsp?locale=zh_TW)，安裝完畢後在 Jenkins 中指向剛剛的目錄。

![jenkins-install-java](./jenkins-install-java.png)

## 初始化

根據安裝時所指定的 Port ，用瀏覽器開啟網站，可以看到 **Unlock Jenkins** 的畫面，到紅字所描述的路徑打開檔案，將裡面的密碼複製貼上。

![unlock-jenkins](./unlock-jenkins.png)

接著要安裝 Plugin，這邊不會建議使用官方推薦的，而改用自行挑選。

![customize-jenkins](./customize-jenkins.png)

裝 Jenkins 的目的是為了要可以編譯 dotnet framework，找到 Build Tools 將 `MSBuild` 打勾，若還有前端則需要把 `NodeJS` 也打勾。

![jenkins-plugin-build-tool](./jenkins-plugin-build-tool.png)

往下看到 Pipelines and Continuous Delivery，整合的目標是 Gitea 因此將 Github 的兩個選項都取消。

![jenkins-plugin-pipeline](jenkins-plugin-pipeline.png)

最後，會建議將 `Locale` 這個套件裝起來，可以強制看到的語系不是根據瀏覽器來決定。

![jenkins-plugin-locale](./jenkins-plugin-locale.png)

都挑選完畢以後，就可以開始安裝啦。

![jenkins-getting-start](./jenkins-getting-start.png)

# 整合 Gitea

都安裝完以後，進入系統，先來到 `管理Jenkins` > `設定系統`，找到 locale 的區塊將預設語言設定為 **en-us**，並將下面核選方塊打勾，儲存後就可以看到系統變成英文版。

> 我會習慣切成英文版是為了要找資料比較方便

![jenkins-locale-setting](./jenkins-locale-setting.png)

## 安裝 Gitea

接著要安裝 Gitea 的[套件](https://plugins.jenkins.io/gitea/)，在 `Manage Jenkins` > `Manage Plugins` > `Available` 搜尋 Gitea，打勾後選擇下方的 `Download now and install after restart`。

![jenkins-plugin-gitea](./jenkins-plugin-gitea.png)

安裝後回到 `Manage Jenkins` > `Configure System`，找到 `Gitea Server` 的區塊，點選 Add，將公司的 Gitea Server 資訊輸入進去。

![jenkins-configure-gitea-server](./jenkins-configure-gitea-server.png)

官方建議勾選 `Manage Hooks` 並使用 Personal Token 的方式，Add > Jenkins 新增 **Credential Provider**，Kind 選擇 `Gitea Personal Access Token`。

![jenkins-credential-add](./jenkins-credential-add.png)

開啟 Gitea 的網站，在 `設定` > `應用程式` 中，產生新的 Token。

![gitea-access-token](./gitea-access-token.png)

複製所產生的 Token 並且回到 Jenkins 畫面貼上，最後按下儲存。

![jenkins-configure-gitea-server-hook](./jenkins-configure-gitea-server-hook.png)

> 建議使用一個公用的帳號，例如: ci，來產生 Token，才可以避免權限相關的問題，也不要有人員異動後需要重新弄得問題。

## 連接 Repo

>  呼~終於要開始整合。

回到 Jenkins 主畫面，點選 `NewItem`，輸入名稱並選擇 `Organization Folder`，按下 OK。

![jenkins-new-item](./jenkins-new-item.png)

移動到 Projects 區塊，選擇 `Gitea Organization`。

![jenkins-org-folder-config](./jenkins-org-folder-config.png)

選擇前面所輸入的 **Gitea Server** 及 **Credentials**，Owner 輸入要納入的組織或是使用者名稱。

> 這邊是用自己作為設定

![jenkins-org-config-gitea-org](./jenkins-org-config-gitea-org.png)

只要前面的所有設定都正確，系統會先掃一次整個 Owner 底下的 repo，檢查是否有 `Jenkinsfile` 檔案。

![jenkins-org-folder-scan](./jenkins-org-folder-scan.png)

## 建立 Jenkinsfile

萬事俱備，只欠東風，現在只要建立一個 Repo，並且加上 `Jenkinsfile` 即可。
在個人使用者底下建立一個新的 Repo 叫做 `JenkinsTest`。

![gitea-create-repo](./gitea-create-repo.png)

在本地開一個資料夾並且新增檔案 `Jenkinsfile` 將下面的內容貼上，並且 commit 後 push 上去。

```jenkinsfile
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```

回到 Jenkins 組織的畫面，直接點選 `Scan Gitea Organization Now`，讓系統重新掃描一次，看到 log 就會有剛剛建立的 repo 並且發現有 Jenkinksfile 的存在。

![jenkins-org-folder-scan-with-repo](./jenkins-org-folder-scan-with-repo.png)

點選左側的 Status，就可以看到 Repo 會在右側的列表中。

![jenkins-org-folder-status](./jenkins-org-folder-status.png)

點進 Repo 可以看到 Branch 及 Pull Request 的狀態。

![jenkins-org-folder-repo-status](./jenkins-org-folder-repo-status.png)

到目前這個狀態，可以確定 Gitea 及 Jenkins 的整合成功!!

> 寫到這邊發現篇幅有點長了，接下來的 jenkinsfile 撰寫，就挪到下一篇囉

# Reference

- [Windows (jenkins.io)](https://www.jenkins.io/doc/book/installing/windows/#invalid-service-logon-credentials)
- [Gitea | Jenkins plugin](https://plugins.jenkins.io/gitea/)
- [Locale | Jenkins plugin](https://plugins.jenkins.io/locale/)