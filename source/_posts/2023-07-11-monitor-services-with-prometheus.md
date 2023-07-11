---
title: 使用 Prometheus 來監控系統
date: 2023-07-11 20:46:44
updated: 2023-07-11 20:46:44
categories:
- DevOps
tags:
- DevOps
- Prometheus
- Grafana
- Monitor
thumbnail: prometheus-hd.jpg
---

身為一個開發者，部屬服務後不是射後不理，應該要去觀察並且監控狀態，**Prometheus** 是一套簡單方便也完整的監控系統，從 Windows/Linux/nginx等等的服資料都可以收集。

<!-- more -->

## 介紹 Prometheus

Prometheus 是一款開放原始碼的監控和警報系統，用於收集 metrics、儲存和分析時間序列資料。採用靈活的資料模型和強大的查詢語言（PromQL），可即時監控各種元件和應用程式。Prometheus 具有自動發現和動態組態的能力，並與其他工具和服務整合緊密，如 Grafana 用於可視化監控資料。

> 更多的介紹我就不說了，就請移駕到 [Overview | Prometheus](https://prometheus.io/docs/introduction/overview/)

下圖是官方所提供的架構圖。

![architecture](https://prometheus.io/assets/architecture.png)

## 資料收集

Prometheus 提供了非常多常見的服務收集器，以下會介紹幾個目前有在用的

> 其他的服務可以到 [官方](https://prometheus.io/docs/instrumenting/exporters/)查詢，裡面超多東西

### windows exporter

用來收集主機的資訊，像是 cpu/memory...etc，跟 `node exporter` 是好兄弟，針對 windows 蒐集的資訊更豐富

安裝方法如下，到 [windows_exporter (github.com)](https://github.com/prometheus-community/windows_exporter) 的 release 下載最新版本 msi

如果什麼都不改的話，可以直接執行 msi 檔案安裝；若要改設定可以參考下面的基本設定

```shell
msiexec /i <path-to-msi-file> ENABLED_COLLECTORS=os,iis LISTEN_PORT=5000
```

- **ENABLED_COLLECTORS**，可以重新指定要收集哪些資訊，若沒有指定的話，預設是 cpu/cs/logical_disk/net/os/service/system/textfile
- **LISTEN_PORT**，指定要監聽的 port，預設是 9182

### cAdvisor

用來收集 Docker Container 的服務狀態

```yaml
version: "3.9"

services:
  # ...其他服務
  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    container_name: cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
```

### nginx exporter

有使用到 nginx 做 reverse proxy 服務的話，可以使用這個 exporter 來監控服務流量的狀況

必須修改 nginx 的設定，把 `stub_status` 打開

```nginx
# for prometheus
server {
    listen 8080;
    location /stub_status{ 
       stub_status on;
    }
}
```

把 nginx 和 exporter 用 docker compose 一起跑起來

```yaml
version: "3.9"

services:
  nginx:
    image: nginx
    restart: always
    container_name: nginx
    ports:
      - 80:80
      - 443:443
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
    environment:
      - TZ=Asia/Taipei
  nginx-exporter:
    image: nginx/nginx-prometheus-exporter:0.10.0
    restart: always
    container_name: nginx-exporter
    ports:
     - 9113:9113
    command:
      - -nginx.scrape-uri
      - http://nginx:8080/stub_status
```

### Gitea metrics

Gitea 本身不需要外掛，自己服務本身就有提供，只需要在設定檔開啟即可，參考官方的設定 [Config Cheat Sheet | Gitea Documentation](https://docs.gitea.com/administration/config-cheat-sheet#metrics-metrics)

> 設定後記得重啟服務

```ini
[metrics]
ENABLED=true
ENABLED_ISSUE_BY_REPOSITORY=true
ENABLED_ISSUE_BY_LABEL=true
```

## 設定Prometheus

把上面想要收的一些 exporter/metrics 都弄好後，就要回到 Prometheus 本身的設定，建立一個 `prometheus.yml` 檔案。
把想要收集的資訊放在 `scrape_configs` 的區塊內，並且根據不同類型做對應的設定

- **scheme** ，預設是 http，可以改成 https
- **metrics_path**，預設是 `/metrics`，大多服務應該不用改變
- **static_configs**，收集資料的目標，可以多台設定
- **scrape_interval**，可以針對單一個 job 有不同的收集尖閣

像下面的範例就是收集了 `gitea`, `windows`, `nginx`, `cadvisor` 四個類型的資訊

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "windows"
    scheme: http
    static_configs:
      # multiple target
      - targets: ["server1:9182","server2:9182","server3:9182"]
  - job_name: 'cadvisor'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
    - targets: ['cadvisor:8080']
```

## 設定警報

### 警報規則

這邊用 windows 作為範例，產生一個 `windows_rule.yml`，監控 cpu/memory/disk 的使用率達到 90 % 以上的時候就要發送警報

- **alert**，警報名稱
- **expr**，使用 PromQL 作為查詢語
- **for**，持續時間
- **labels**，標籤類型
- **annotations**，訊息標題和描述

> 對於規則的定義，一開始如果不知道怎麼定義，可以問 ChatGPT 讓他先提供一些範例再來做調整

```yaml
groups:
- name: Windows_Resource_Alerts
  rules:
  - alert: High_CPU_Usage
    expr: 100 * (1 - avg by (instance) (irate(windows_cpu_time_total{mode="idle"}[1m]))) > 90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: High CPU usage on {{ $labels.instance }}
      description: The average CPU usage is above 90% for the past 1 minutes.

  - alert: High_Memory_Usage
    expr: 100 * (1 - avg by (instance) (windows_os_physical_memory_free_bytes / windows_cs_physical_memory_bytes)) > 90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: High memory usage on {{ $labels.instance }}
      description: The available memory is below 10% for the past 5 minutes.

  - alert: High_Disk_Usage
    expr: 100 * (1 - avg by (instance, volume) (windows_logical_disk_free_bytes{volume!~"HarddiskVolume."} / windows_logical_disk_size_bytes{volume!~"HarddiskVolume."})) > 90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: High disk usage on {{ $labels.instance }}
      description: The available disk space is below 10% for the past 5 minutes.
```

### 警報管理

設定完規則，接著要設定發送到哪，用 discord 作為範例(雖然[官方](https://prometheus.io/docs/alerting/latest/configuration/#receiver)沒有列出來，但實際上是可以運行)，新增一個檔案 `alertmanager.yml` 並設定內容。

```yaml
global:
  resolve_timeout: 5m
route:
  group_by: ['alertname', 'job']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 3h
  receiver: discord
receivers:
- name: discord
  discord_configs:
  - webhook_url: "https://discord.com/api/webhooks/xxxxxx/xxxxx"
```

### 設定Prometheus

接著要在 `prometheus.yml` 加上下面這組設定，有多個規則檔案的話在自行往下新增；同時也增加 AlertManagement 的設定。

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "windows_rules.yml"
```

## 完整實作

### Docker版

根據前面所提到的部份，提供一份完整的 `docker-compose.yml` 設定

```yaml
version: "3.9"

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus:/etc/prometheus
    environment:
      - TZ=Asia/Taipei
    ports:
      - 9090:9090
  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    container_name: cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
  alertmanager:
    image: quay.io/prometheus/alertmanager
    container_name: alertmanager
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
```

以及 `prometheus.yml` 設定

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "windows_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "windows"
    scheme: http
    static_configs:
      # multiple target
      - targets: ["server1:9182","server2:9182","server3:9182"]
  - job_name: 'cadvisor'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
    - targets: ['cadvisor:8080']
```

資料夾的結構會是這樣

![docker-compose-folder](docker-compose-folder.png)

### Window安裝版

如果真的需要使用 exe 執行的方法，也可以到 [Download | Prometheus](https://prometheus.io/download/) 下載對應的程式，再把執行程式註冊到 `服務` 即可

> 記得把前面所設定的 yaml 檔案放進去

![windows-folder](windows-folder.png)

## Reference

- [Overview | Prometheus](https://prometheus.io/docs/introduction/overview/)
- [使用 Prometheus 和 Grafana 打造 Flask Web App 監控預警系統 (techbridge.cc)](https://blog.techbridge.cc/2019/08/26/how-to-use-prometheus-grafana-in-flask-app/)
- [iT 邦幫忙::一起幫忙解決難題，拯救 IT 人的一天 (ithome.com.tw)](https://ithelp.ithome.com.tw/m/articles/10299053)
- [google/cadvisor: Analyzes resource usage and performance characteristics of running containers. (github.com)](https://github.com/google/cadvisor)
- [nginxinc/nginx-prometheus-exporter: NGINX Prometheus Exporter for NGINX and NGINX Plus (github.com)](https://github.com/nginxinc/nginx-prometheus-exporter)
