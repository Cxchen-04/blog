---
title: Docker/Kubernetes内网生产环境导致镜像无法拉取
date: 2025-05-24 15:17:08
tags:
---
docker因为墙的问题很好解决，这篇主要是关于企业内网自建1.24版本之后的k8s集群，containerd无法拉取镜像的问题。

### Docker
由于2024年6月开始国内的大量docker镜像停服😤，Docker无法下载安装镜像无法拉取😭 详情我在[docker](https://www.luelulu.com/2025/03/25/docker/#%E8%A7%A3%E5%86%B3%E5%9B%BD%E5%86%85Docker%E9%95%9C%E5%83%8F%E9%97%AE%E9%A2%98)篇已经写过。这里就不再赘述。
### K8s Containerd的运行时下
由于k8s在1.24版本弃用了docker的支持 所以更支持用containerd来跑容器，可是在我们在创建pod以及各种组件containerd都会默认去从dockerhub去拉取镜像，在这里不论我去containerd的config.toml文件下添加镜像源也不管事儿，拉取镜像都是失败。所以我直接用梯子的方式给我的内网集群都添加上了代理。

可是在添加上全局代理之后，我的k8s还是拉不到镜像， 因为Kubernetes 的容器拉取行为由 容器运行时（如 containerd、Docker） 执行，它们运行在系统服务中，并 不继承你 shell 中的代理环境变量（如 ALL_PROXY）。
由于我的代理是用shadowsocks协议 是另一台海外的服务器自建的vpn隧道，我直接在本地的各个节点下载好客户端就行。
接下来 我们开启完整的操作步骤 让你的内网自建k8s集群也能轻松访问外网拉取到镜像。

第一步：在公司服务器上安装 shadowsocks-libev（作为客户端）
```bash
sudo apt update
sudo apt install shadowsocks-libev -y
# centos 就替换成 yum
```
第二步：配置 Shadowsocks 客户端
```bash
sudo nano /etc/shadowsocks-libev/config.json
```
填入如下内容(改成你自己的)：
```json
{
    "server":"123.123.123.123",        // 你香港ECS的公网IP
    "server_port":8388,                // SS端口
    "local_port":1080,                 // 本地监听端口（默认1080）
    "password":"your_password",        // SS密码
    "timeout":300,
    "method":"aes-256-gcm",            // 加密方式，需与你的SS服务端一致
    "fast_open": false
}
```
第三步：启动 Shadowsocks 客户端
```bash
sudo systemctl restart shadowsocks-libev-local
sudo systemctl enable shadowsocks-libev-local
```
如果显示notfound话 那就代表没有守护进程 这里我们需要配置下守护进程
1. 我们用的是 ss-local 命令（客户端，绑定本地 127.0.0.1:1080）
```bash
which ss-local 
/usr/bin/ss-local   # 复制这个
# 如果找不到，请先安装
sudo apt install shadowsocks-libev
```
2. 创建 systemd 服务文件：
```bash
sudo nano /etc/systemd/system/ss-local.service
```
填入以下内容（注意配置文件路径与你实际一致）：
```ini
[Unit]
Description=Shadowsocks Local Client Service
After=network.target

[Service]
ExecStart=/usr/bin/ss-local -c /etc/shadowsocks-libev/config.json
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
如果 ss-local 路径不是 /usr/bin/ss-local，请用 which ss-local 替换它。
3. 重新加载 systemd 并启动服务：
```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload

sudo systemctl start ss-local.service
sudo systemctl enable ss-local.service
# 查看状态确认是否启动成功
sudo systemctl status ss-local.service
```
✅ 启动成功后，你的本地 127.0.0.1:1080 就可以作为 socks5 代理使用了：
```bash
curl --socks5-hostname 127.0.0.1:1080 https://google.com
```
🧪 附：测试 IP 是否成功通过代理
```bash
curl --socks5-hostname 127.0.0.1:1080 cip.cc
```
第四步：使用 Privoxy 转换 SOCKS5 → HTTP
这里你会发现containerd还是拉取不了镜像，这里我们就需要一个Privoxy工具将SOCKS5→HTTP

安装并配置privoxy
```bash
sudo apt install privoxy -y
# 配置 Privoxy 将 HTTP 请求转发给 ss-local 的 SOCKS5 端口
sudo nano /etc/privoxy/config
```
添加一行（放到文件最后）：
```conf
forward-socks5t / 127.0.0.1:1080 . 
```
启动并检查端口监听
```bash
sudo systemctl restart privoxy
sudo systemctl enable privoxy
# 检查监听：
ss -tuln | grep 8118
# 你应该能看到：
127.0.0.1:8118
```
第五步：为容器运行时配置代理
1. 创建或编辑代理配置文件（containerd 使用 systemd 启动）：
```bash
sudo mkdir -p /etc/systemd/system/containerd.service.d
sudo nano /etc/systemd/system/containerd.service.d/http-proxy.conf
```
2. 内容如下（请改为你实际使用的代理端口）：
```ini
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:8118"
Environment="HTTPS_PROXY=http://127.0.0.1:8118"
Environment="NO_PROXY=127.0.0.1,localhost,10.0.0.0/8,192.168.0.0/16"
```
🧠 注意：
•	containerd 不能直接识别 SOCKS5，所以需要用一个额外的本地 HTTP 代理转接 SOCKS5，或者让 ss-local 启动一个 http-local 端口


这样我们就可以直接用containerd来拉取镜像了！