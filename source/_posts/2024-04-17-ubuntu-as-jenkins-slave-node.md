---
title: 使用 Ubuntu 來作為 Jenkins 的 Slave Node
date: 2024-04-17 10:00:10
updated: 2024-04-17 10:00:10
categories:
- CICD
- Jenkins
tags:
- Jenkins
- CICD
thumbnail: 
---

本篇會介紹如何使用 Ubuntu 來作為 Jenkins Slave Node 並且採用 SSH 的方式做連線。

<!-- more -->

> 相關環境版本
> Jenkins Server: windows 11
> Jenkins Version: 2.440.2
> Slave Node: Ubuntu 22.04

## Slave Node

安裝 `openssh server`

```shell
sudo apt-get install -y openssh-server
```

### 新增使用者

> 不能用安裝時所產生的使用者作為 ssh 連線的帳號

```shell
sudo adduser jenkins
```

產生 ssh key

```shell
ssh-keygen
```

在 `~/.ssh` 的目錄下會看到剛剛所產生的檔案，預設會是 `id_rsa(私鑰)` 及 `id_rsa.pub(公鑰)`

複製公鑰 `id_rsa.pub` 至另一個檔案 `authorized_keys`，用來給 Jenkins 驗證對答案

```shell
cat id_rsa.pub > authorized_keys
```

## Jenkins Server

在 server 的 .ssh 資料夾中新增 `know_hosts` 檔案，並加上前面所產生的 `id_rsa.pub`

> 如果是 windows ，可以參考安裝 [SSH server](https://learn.microsoft.com/zh-tw/windows-server/administration/openssh/openssh_install_firstuse?tabs=powershell)

## 新增 Node

到 Jenkins 網站的 Nodes 頁面中新增一個 Node

![new-node](new-node.png)

- Remote root directory，輸入遠端的執行跟目錄，建議是登入的使用者底下 `/home/{user}/jenkins_agent`

- Labels，根據 job 的需要做輸入

- Launch method，選擇 `Launch agents vis ssh`

  - Host，輸入目標電腦的 ip
  - Credentials，用新增的方法來產生一組專門資料，詳細作法後面介紹
  - Host Key Verification Strategy，選擇 `Non verifying Verification Strategy`

  > 只測試到這個做法可以順利將 node 跑起來

>  如果 server 和 slave 是不同平台，例如 windows/linux，那就必須在下面的 `Node Properties` 區塊中，重新設定一些 `Tool Locations`

### 建立 Credential

在 Kind 選擇 `SSH Username with private key` 的方式做認證

![add-credential](add-credential1.png)

- Username，輸入登入 node 的帳號
- Private Key，將前面 slave node 步驟所產生的 `id_rsa` 內容貼上

![add-credential](add-credential2.png)

### 啟動

以上設定都完成後，就可以讓 Node 啟動，可以發現連線成功，但是啟動失敗

```
[04/17/24 11:18:18] [SSH] Authentication successful.
[04/17/24 11:18:19] [SSH] Checking java version of /home/jenkins/jenkins-agent/jdk/bin/java
Couldn't figure out the Java version of /home/jenkins/jenkins-agent/jdk/bin/java
bash: line 1: /home/jenkins/jenkins-agent/jdk/bin/java: No such file or directory
```

到這邊已經差不多成功，剩下一些軟體的安裝

## 安裝相關軟體

### 安裝 java

```shell
sudo apt install -y openjdk-17-jre
```

### 安裝 git

```shell
sudo apt install -y git
```

### 安裝 docker

參考 [Install Docker Engine on Ubuntu | Docker Docs](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

```shell
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

To install the latest version, run:

```shell
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Permssion Denied

當有看到以下的錯誤，可以用底下的做法來調整

> Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get https://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json: dial unix /var/run/docker.sock: connect: permission denied

Creating a group - docker

```bash
sudo groupadd docker
```

Adding user to the group "docker"

```bash
sudo usermod -aG docker $USER

sudo chmod 666 /var/run/docker.sock
```



以上都安裝完畢，只要回到剛剛新增的 Node 並且重新啟動，就可以正常啟用。