---
title: Ansible 初體驗
date: 2026-08-03 13:59:17
updated: 2026-08-03 13:59:17
categories:
- Infra
tags:
- IaC
- Infra
thumbnail:
---

聽過 Ansible 這個工具很久，但一直都不知道怎麼用，趁著最近比較有空，加上有 AI 這麼好用的工具，來好好的學習一下，並且看看怎麼套用到自己的工作上。

<!-- more -->

## 為何選擇

在電腦上執行某些特定的指令這件事情是平常稀鬆的，例如說要到遠端主機上改某個檔案，然後讓服務重啟，這樣的步驟每次一直做也很煩，而且太多人為介入，怎麼知道有沒有操作失誤。

這時候會想到的是，那是不是可以把會執行的指令整理起來，用 shell 的方式來處理，會想到這個步驟，就已經是有自動化的概念。

那如果有多台電腦要執行，shell script 能不能處理，一定可以，但好像會比較麻煩一點，會希望有個更簡單一點的工具來協助，這時候 Ansible 就可以派上用場。

當然 Ansible 不單是多台電腦的執行好用，他還有其他的好處，例如: 只要在 client 端安裝環境， server 端只要有支援 ssh 就可以，而且沒有限定 os 平台，windows 也有支援。

和 shell script 更大的不同是透過 yaml 來描述機制該做什麼事情，~~一起成為 YAML 工程師吧~~。

## 架構和名詞介紹

```
你的電腦(控制機,裝 Ansible)
 └─ ssh ──→ 主機 A(什麼都不用裝)
 └─ ssh ──→ 主機 B(什麼都不用裝)
```

基本上可以分為兩種檔案組合成完整的指令

### Inventory

主機清單和變數相關的描述設定，總共有三臺主機，兩個群組 web、ws，一個變數

```yaml
all:
  vars:
    ansible_user: deploy
  children:
    web:
      hosts:
        WEB01:
        WEB02:
    ws:
      hosts:
        WS01:
```

### Playbook

描述要執行的劇本(腳本)

```yaml
- hosts: web
  tasks:
    - name: 步驟一
      ...
    - name: 步驟二
      ...
```

### Task

每個 task 都是一個步驟通常是一個指令

```yaml
- name: 建立資料夾           # 給人看的說明
  ansible.builtin.file:      # 模組名(誰出的.哪一包.模組)
    path: /tmp/demo          # 參數
    state: directory         # 參數
```

### 執行結果

同一份劇本跑兩次，結果一樣、不會壞 (冪等 Idempotent)。每個 task 執行完只有四種結果:

| 結果          | 意思                  |
| :------------ | :-------------------- |
| `ok`(綠)      | 現況已符合,什麼都沒做 |
| `changed`(黃) | 動手改了,現在符合了   |
| `failed`(紅)  | 做不到,停止           |
| `skipped`(藍) | 條件不成立跳過        |

## 動手做

在公司新增一個服務，就可能要去設定 nginx 一次，這樣的操做就很適合用 ansible 來處理

### 第一版

inventory

```yaml
all:
  vars:
    ansible_user: "{{ lookup('env', 'SSH_USER')}}"
    ansible_password: "{{ lookup('env', 'SSH_PASS')}}"
    ansible_become_password: "{{ lookup('env', 'SSH_PASS')}}"
  children:
    web:
      hosts:
        WEB01:
        WEB02:
```

playbook

```yaml
- name: 更換 nginx config 並且重新載入
  hosts: all
  become: true
  tasks:
    - name: 備份檔案
      ansible.builtin.command:
        cmd: mv /etc/nginx/conf.d/main.conf /etc/nginx/conf.d/main.conf.bak
    - name: 複製本機的檔案到主機
      ansible.builtin.copy:
        src: ./../main.conf
        dest: /etc/nginx/conf.d/main.conf
    - name: 執行測試
      ansible.builtin.command:
        cmd: nginx -t
      register: ngx
      changed_when: false
      failed_when: ngx.rc != 0 or 'syntax is ok' not in ngx.stderr
    - ansible.builtin.debug:
        msg: "{{ ngx.stderr_lines }}"
    - name: 重新載入
      ansible.builtin.command:
        cmd: nginx -s reload
```

### 第二版

這樣的步驟會有什麼問題嗎? 每次執行的時候都會做備份然後在複製過去然後重新載入，聽起來超完美，當檔案沒有修改的時候也會這樣做就不夠完美了

請 AI 給出建議以後優美很多也很好懂
將第一和第二步驟合併成一個，只有在檔案不一樣的時候會備份

```yaml
- name: 更換 nginx config 並且重新載入
  hosts: all
  become: true
  tasks:
    - name: 複製本機的檔案到主機並備份檔案
      ansible.builtin.copy:
        src: ./../main.conf
        dest: /etc/nginx/conf.d/main.conf
        backup: true
    - name: 執行測試
      ansible.builtin.command:
        cmd: nginx -t
      register: ngx
      changed_when: false
      failed_when: ngx.rc != 0 or 'syntax is ok' not in ngx.stderr
    - ansible.builtin.debug:
        msg: "{{ ngx.stderr_lines }}"
    - name: 重新載入
      ansible.builtin.command:
        cmd: nginx -s reload
```

### 第三版

AI 更進一步的提出了使用 handler 的做法，讓整個架構更明確

```yaml
- name: 更換 nginx config 並且重新載入
  hosts: all
  become: true
  tasks:
    - name: 複製本機的檔案到主機並備份檔案
      ansible.builtin.copy:
        src: ./../main.conf
        dest: /etc/nginx/conf.d/main.conf
        backup: true
      notify: reload nginx
    - name: 執行測試
      ansible.builtin.command:
        cmd: nginx -t
      register: ngx
      changed_when: false
      failed_when: ngx.rc != 0 or 'syntax is ok' not in ngx.stderr

  handlers:
    - name: reload nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
```

## 心得

練習到後來，在想要怎麼實際應用上 production 主機的時候，卡在要過跳板機才能連到遠端主機，就想到說其實可以放在 action workflow 中，那這樣有沒有用 ansible 也沒那麼大的差異，但在仔細思考以後發現還是有差異，用 action 就只有在 event trigger 的時候使用，用 ansible 的話我就可以在自己電腦做測試執行，不用說等上了 action 才能確認，而且這些 script 也可以給其他同事使用，那就降低了不少負擔。