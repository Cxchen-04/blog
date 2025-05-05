---
title: Prometheus+Grafana
date: 2025-03-30 13:43:43
tags: [运维,监控]
categories: [运维,监控,Prometheus+Grafana]
cover: /images/监控.png
---
## 简介
#### 🔍 Prometheus 是什么？

Prometheus 是一个“监控和数据采集系统”，简单理解就是一个“收集数据的仓库+大脑”。

📦 它主要做这些事：
	•	采集数据：比如服务器的 CPU 使用率、内存占用、网络流量、请求次数等。
	•	存储数据：把这些数据以时间序列的形式存到自己的数据库里。
	•	查询数据：支持用专门的 PromQL 语言查询、筛选、聚合各种指标。
	•	设置告警：可以设置规则，当某个指标异常时触发告警（例如 CPU 连续5分钟超过 90%）。

👀 简单说，它是“采集、存、查、报”的一套系统，不负责可视化。

#### 📊 Grafana 是什么？

Grafana 是一个数据可视化工具，用来“把数据变成图表”。

🎨 它主要做这些事：
	•	连接数据源：比如连接 Prometheus，读取里面的指标数据。
	•	绘制图表和仪表盘：你可以用它制作漂亮的图、线、饼图、热图等各种形式。
	•	设置仪表盘：将不同的数据组合成统一的监控面板，便于一眼看出系统健康状况。
	•	告警通知：Grafana 也支持设置告警（但数据还是从 Prometheus 来）。

👀 简单说，它是“读数 + 画图 + 展示”的一套系统，不负责采集和存储。

## 安装
#### 安装Node Exporter（监控服务器性能）
Node Exporter是Prometheus的官方工具，可以帮助采集服务器的硬件指标（CPU、内存、磁盘等）。

``` bash Ubuntu
ssh root@你的服务器ip

sudo useradd --no-create-home --shell /bin/false node_exporter
# 创建node_exporter用户(安全起见)

wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-1.7.0.linux-amd64.tar.gz
sudo cp node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
# 下载并解压node_exporter

sudo nano /etc/systemd/system/node_exporter.service
# 设置systemd启动服务
```
写入以下内容
``` bash Ubuntu
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter #文件路径

[Install]
WantedBy=default.target
```
启动并设置开机启动
``` bash Ubuntu
sudo systemctl daemon-reexec
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```
测试一下：
打开浏览器访问：
http://你的服务器IP:9100/metrics
如果能看到一堆指标文字，说明成功！

#### 安装Prometheus
下载Prometheus：
``` bash Ubuntu
wget https://github.com/prometheus/prometheus/releases/download/v2.50.0/prometheus-2.50.0.linux-amd64.tar.gz
tar xvf prometheus-2.50.0.linux-amd64.tar.gz # 解压
cd prometheus-2.50.0.linux-amd64 
# 进入文件夹里面创建.yml文件

nano prometheus.yml # 创建prometheus配置文件
```
写入一下内容：
``` Yaml
global:         # 一定要写正确格式，包括空格以及重复定义
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs: # 这个可能会有冲突 
      - targets: ['localhost:9100'] 

  - job_name: 'website_http_check'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - http://你的域名或者IP  # 这里就是grafana的监控项
          - prometheus:9090
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115  # blackbox_exporter 监听端口
```

#### 安装Blackbox Exporter(探测网站状态)
``` bash Ubuntu
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar xvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
cd blackbox_exporter-0.25.0.linux-amd64 
./blackbox_exporter     # 进入文件夹启动
# 也可以添加systemd启动服务，和node_exporter类似。
```

#### 启动prometheus
``` bash Ubuntu
./prometheus --config.file=prometheus.yml
```
访问浏览器：
http://你的服务器IP:9090，你应该可以看到 Prometheus 的 Web 页面。

#### 📊安装 Grafana
``` bash
sudo apt-get install -y apt-transport-https software-properties-common
# 安装依赖
 
sudo apt-get install -y wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt-get update
# 添加grafana仓库

sudo apt-get install grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
# 安装并启动grafana
```
访问grafana：
浏览器打开 http://你的服务器IP:3000，默认账号密码是：
用户名：admin
密码：admin  登录之后会要求修改密码

#### 📈配置 Grafana
添加数据源：选择 Prometheus，填入 http://localhost:9090
导入 Dashboard 模板：
Node Exporter：导入模板 ID 1860
Blackbox 网站状态：导入模板 ID 7587



## 使用

### grafana更改中文
要将Grafana配置为中文，你可以参照以下步骤进行操作
1. 打开grafana的默认配置文件：/opt/bitnami/grafana/conf/defaults.ini
                          /usr/share/grafana/conf/defaults.ini
2. 在该文件中，找到default_language这一行，将en-US改为zh-Hans。这样grafana的语言就会更改为中文。
3. 保存并关闭文件
4. 启服务刷新网页即可
``` bash
systemctl restart grafana-server
```

### 监控告警
#### 🛠️ 设置 Prometheus 告警规则
Prometheus 支持基于查询的告警规则，你可以通过 PromQL 设置条件，比如当系统负载过高时发出告警。
1. 创建告警规则文件
在 Prometheus 配置目录下创建一个 alert.rules 文件（如果没有的话）：
``` bash Ubuntu
nano /etc/prometheus/alert.rules 
# 看你prometheus安装/解压到了哪里
```
2. 在文件中加入告警规则，例如：
```  Yaml
groups:
  - name: node_alerts
    rules:
    - alert: HighCPUUsage 
      expr: 100 * (1 - avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[1m]))) > 80
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "CPU usage is above 80% on {{ $labels.instance }}"
        description: "CPU usage is above 80% for 5 minutes on {{ $labels.instance }}."
        
    - alert: HighLoadAverage
      expr: node_load1 > 2
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High load average on {{ $labels.instance }}"
        description: "The 1-minute load average is above 2 on {{ $labels.instance }}."
```
HighCPUUsage：当 CPU 使用率超过 80% 持续 5 分钟时，触发告警。
HighLoadAverage：当系统 1 分钟平均负载超过 2 时，触发告警。
2. 修改 Prometheus 配置文件，加载告警规则
编辑 Prometheus 配置文件 prometheus.yml，加入规则文件：
``` bash Ubuntu
rule_files:
  - "/etc/prometheus/alert.rules"
```
3. 重新启动 Prometheus

修改完配置后，重新启动 Prometheus 服务来应用告警规则：
``` bash
sudo systemctl restart prometheus
```
#### 🛠️配置 Grafana 告警通知

一旦 Prometheus 设置了告警规则，你就可以在 Grafana 中查看告警并设置通知。

1. 设置通知渠道

首先需要在 Grafana 配置一个通知渠道，常用的有 邮件、钉钉、Slack 等。

配置邮件通知
	1.	打开 Grafana UI，进入 “Alerting” > “Notification channels”。
	2.	点击 “Add channel”。
	3.	选择通知方式（比如 Email）并填写相关信息：
	•	Email Address：收件人邮箱
	•	SMTP：配置发送邮件的 SMTP 服务（如 Gmail 或自己配置的 SMTP）

配置钉钉/Slack 通知

你可以选择使用钉钉或 Slack 的 Webhook，步骤类似，直接填写 Webhook URL。

2. 配置告警规则

在 Grafana 中，进入你要监控的 Dashboard，并进行告警设置：
	1.	打开需要设置告警的图表，点击图表右上角的 “Alert” 标签。
	2.	配置告警条件，比如：
	•	当系统负载超过某个阈值。
	•	配置告警时间（比如 5 分钟内触发）。
	3.	在 “Notifications” 部分，选择之前配置好的通知渠道（比如 Email、钉钉等）。

3. 保存告警设置

设置完成后，点击 “Save” 来保存告警规则。Grafana 会根据你设置的条件，定期检查，并在触发告警时通过邮件或钉钉等渠道发送通知。
#### 🧑‍💻验证告警是否有效
1. 要让 Grafana 能通过 SMTP 发送邮件告警通知，你需要配置它的 smtp 邮件发送服务。以Gmail为例
  
| 配置项         | 示例（以 Gmail 为例）              |
|----------------|-------------------------------------|
| SMTP 服务器地址 | smtp.gmail.com                      |
| 端口号         | 587 (TLS) / 465 (SSL)               |
| 发件邮箱       | yourname@gmail.com                  |
| 发件邮箱密码   | Gmail 生成的“应用专用密码”         |
| 接收人邮箱     | 任意你想收到告警的人邮箱地址       |

1. 修改 Grafana 配置文件 grafana.ini
默认路径通常是：
``` bash
nano /etc/grafana/grafana.ini
```
找到以下配置段落，把它修改成你的信息（注意取消注释）：
``` bash
#################################### SMTP / Emailing ##########################
[smtp]
enabled = true
host = smtp.gmail.com:587
user = yourname@gmail.com
password = your_app_password   ; # Gmail 需要使用“应用专用密码”而不是登录密码
;cert_file =
;key_file =
skip_verify = false
from_address = yourname@gmail.com
from_name = Grafana Monitor

[emails]
;welcome_email_on_sign_up = false
```
💡如果你使用的是 QQ 邮箱、阿里邮箱、163 等都可以，改下 host、端口、密码即可
3. 🔐 关于 Gmail 的“应用专用密码”：
	1.	登录你的 Gmail 账号
	2.	打开： https://myaccount.google.com/security
	3.	开启 两步验证
	4.	找到“应用专用密码”，创建一个密码用于 Grafana 发送邮件
	5.	拷贝该密码，贴到上面 password 一栏中
4. 🔄重启 Grafana 服务
配置改好之后需要重启 Grafana 才能生效：
``` bash
sudo systemctl restart grafana-server
```
1. ✉️在 Grafana 中测试发送邮件
	1.	登录 Grafana → 左侧点击 “Alerting” → “Contact points”
	2.	创建一个新的 Email 通知方式
	3.	输入你希望接收告警的邮箱地址
	4.	保存后点击 “Send test notification” 来验证是否能收到邮件

✅ 如果设置成功，你应该会在邮箱里看到一个测试邮件！

🎉 完成！

## Docker 部署
我们如果要用docker部署的话 需要额外下载一个组件 __"nginx-prometheus-exporter"__ 因为docker的容器有隔离机制 因此我们需要这个nginx容器暴露出来的 __stub_status__  页面

| 组件名             | 作用说明                               | 镜像名称（Docker Hub）                  | 备注                           |
|:-------------------|:---------------------------------------|:----------------------------------------|:-------------------------------|
| Prometheus         | 监控数据采集和存储                     | prom/prometheus                         | 采集 nginx-exporter、node-exporter、blackbox-exporter |
| Grafana            | 数据展示和可视化面板                   | grafana/grafana                         | 连接 Prometheus 做图表 |
| nginx-prometheus-exporter | 抓取 Nginx 的内部状态指标             | nginx/nginx-prometheus-exporter         | 需要 Nginx 开启 stub_status |
| node-exporter      | 抓取服务器本身 CPU、内存、磁盘、网络指标 | prom/node-exporter                      | 服务器基础性能监控 |
| blackbox-exporter  | 外部探测 HTTP/HTTPS 可用性、状态等      | prom/blackbox-exporter                  | 检测网站存活、SSL证书有效期等 |
| Nginx（你的Web服务器） | 提供网站服务+反向代理                  | nginx:latest                            | 被监控的对象 |

### Docker Compose的配置方法
1. Prometheus
docker-compose的配置方法
```yaml
services:
    prometheus:
      image: prom/prometheus
      container_name: prometheus
      ports:
        - "9090:9090"
      volumes:
        - "./prometheus.yml:/etc/prometheus/prometheus.yml"
      restart: always
```
2. Grafana
docker-compose的配置方法
```yaml
services:
    grafana:
      image: grafana/grafana
      container_name: grafana
      ports:
        - "3000:3000"
      restart: always
      environment:
        - GF_SERVER_ROOT_URL=https://yourServername/grafana/
```
3. nginx-prometheus-exporter 暴露容器内部
docker-compose的配置方法。
```yaml
nginx-exporter:
      image: nginx/nginx-prometheus-exporter
      container_name: nginx-exporter
      ports:
        - "9113:9113"
      command:
#        - nginx.scrape-uri=http://nginx/stub_status
        - "--nginx.scrape-uri=https://yourServername/stub_status" # 如果你的网站没有ssl证书 改成http即可
#        - "--nginx.ssl-verify=false"
      depends_on:
        - nginx
      restart: always
```

4. node-exporter 监控服务器性能
docker-compose的配置方法。
```yaml
services:
   nodexport:
      image: prom/node-exporter
      container_name: exporter
      ports:
        - "9100:9100"
      restart: always
```
5. blackbox-exporter 监控网站状态
docker-compose的配置方法
```yaml
services:
    blackbox-exporter:
      image: prom/blackbox-exporter:latest
      container_name: blackbox-exporter
      ports:
        - "9115:9115"
      restart: always
```
