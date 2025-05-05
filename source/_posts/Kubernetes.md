---
title: Kubernetes / K8s
date: 2025-04-13 15:19:50
tags: [运维,k8s]
categories: [运维, k8s]
cover: /images/k8s.png
---
## 简介
Kubernetes（简称 K8s）是 Google 开源的容器编排平台，它的核心能力是：

__自动管理大规模容器：运行、扩容、重启、故障转移、服务发现等__

它的目标是：“你只管说‘我需要几个服务’，K8s 来帮你部署、分配、调度、维护”。

### 开始前
1. 学习 Kubernetes 前你需要会的
   
| 必备知识                  | 状态  |
|---------------------------|-------|
| Linux 基本操作            | ✅    |
| Docker（容器镜像、run、volume） | ✅    |
| CI/CD 简单概念            | ✅    |
| Nginx、Web 服务部署       | ✅    |
| YAML 配置文件             | ✅    |

🥇 入门阶段：了解核心概念（可以 2 小时搞懂）

| 核心概念     | 简单理解                                                         |
|--------------|------------------------------------------------------------------|
| Pod          | 容器的最小运行单位，1~n 个容器组成                               |
| Node         | 工作节点（本质就是服务器）                                      |
| Deployment   | 声明你要运行哪些 Pod、多少个副本、怎么升级                      |
| Service      | 网络服务，提供访问入口（ClusterIP / NodePort）                 |
| Namespace    | 项目隔离空间（类似 Git 分支）                                   |

🥈 实操阶段：本地部署 K8s，跑个服务

| 方法               | 推荐工具                                                    |
|--------------------|-------------------------------------------------------------|
| 本地单节点测试     | minikube ✅ 推荐（轻量 + 官方）                              |
| Docker Desktop 内置 K8s | ✅ Mac 用户也适合                                       |
| 使用 kubectl 控制集群 | 必备 CLI 工具                                             |
| 跑一个 Nginx       | 创建 Deployment + Service                                   |
| 跑你自己的博客镜像 | 比如用你已有的静态 HTML 做一个 nginx 容器跑起来             |

🥉 进阶阶段：部署 + 高可用 + CI/CD 集成

| 技术点                 | 应用                                                       |
|------------------------|------------------------------------------------------------|
| Helm                   | K8s 的 “包管理器”，一键部署 WordPress / Redis / MySQL     |
| Ingress                | HTTP 代理路由入口，比如实现 blog.karen.com               |
| ConfigMap + Secret     | 配置注入（如博客 config）                                 |
| Kustomize              | 多环境配置管理                                             |
| GitOps                 | CI/CD 自动部署到 K8s（ArgoCD、Flux）                      |
| Prometheus + Grafana   | 集群监控可视化                                             |

### Cluster 集群
1. 一句话理解什么是“集群”（Cluster）
集群 = 一组“联网协作的服务器”，它们对外表现成“一个整体”来提供服务。
__❓集群是不同地区的主机吗？__
__不一定。集群可以在同一机房，也可以跨地区，只要网络能通就可以形成集群。__
但大多数企业为了性能，会把集群部署在同一个局域网 / 数据中心里（比如阿里云、K8s 云服务集群）
__❓集群是“一个服务用一群主机来跑”吗？__
    ✅ 正解！
    你可以：
        •	把一个服务部署到多个节点上，分摊压力（比如你的博客 + 负载均衡）
        •	把多个服务（博客、评论、数据库）部署到同一个集群，资源复用
    Kubernetes 会自动决定：
        •	哪台机器资源够？就分配上去
        •	哪个服务需要 3 个副本？我自动帮你部署到 3 台节点
    	•	哪个节点宕机？自动重启到别的节点上！

2. 📦 类比解释一：集群就像一个“厨房团队”

•	你想点一份“大餐服务”（比如部署你博客 + 数据库 +评论系统）
•	一个厨师（单台服务器）可能忙不过来
•	所以你请了一整组厨师（多台机器）
•	厨房团队（集群）会自动决定：
•	谁煮主菜？
•	谁负责炒菜？
•	万一一个厨师倒下（宕机），谁来顶上？

✅ 这整个厨房团队 = “集群”
✅ 他们之间的分工、调度、故障切换 = Kubernetes 来负责

3. 📌 技术一点说：

| 名称                     | 含义                                                         |
|--------------------------|--------------------------------------------------------------|
| 集群 (Cluster)           | 一组通过网络连接的“节点”服务器                              |
| 节点 (Node)              | 集群中的一台机器，可以是虚拟机或物理机                      |
| 主节点 (Master)          | 管理整个集群的大脑，负责调度、监控、API                     |
| 工作节点 (Worker)        | 实际跑你应用的机器（比如跑 nginx / 博客）                   |
#### k8s中的“集群结构图”

``` bash
             [ Master 控制节点 ]
            ┌───────────────┐
            │  调度服务 kube-scheduler
            │  API Server
            │  状态控制器
            └───────────────┘
                      ↓
       ┌──────────────┬──────────────┐
       ↓              ↓              ↓
 [Worker Node1]   [Worker Node2]  [Worker Node3]
   └─Pod: Nginx     └─Pod: Blog     └─Pod: Redis
   └─Pod: Mongo     └─Pod: Hexo     └─Pod: 评论系统
```
__现实中的“集群”都用在哪？__

| 场景             | 应用                          |
|-----------------|--------------------------------------|
| 大公司网站         | 多地部署，形成全球K8s集群(CDN+服务节点) |
| 企业SaaS平台      | 用集群提升高可用性+自动扩容能力 |
| 云服务商          | 用户只用申请一个“集群”，背后全自动   ｜

##### 集群是多台主机组成的“一个大脑 + 多个手”的团队。K8s 负责调度大脑，让每个服务都能稳定运行，掉了就补、慢了就扩。


## 安装K8s 生产环境
在安装 Kubernetes（特别是用 kubeadm 搭建生产级多节点集群）之前，确实需要做一系列“打地基”的设置，不然后续很容易出现各种诡异问题（如 kubelet 启动失败、网络不通、无法加入集群等）。
### 前提
##### 一、系统基础配置（所有节点都需要）
| 设置项       | 建议值/动作                          | 说明                                   |
|--------------|---------------------------------------|----------------------------------------|
| 操作系统     | Ubuntu 20.04+ / CentOS 7+             | 兼容最好，建议使用 LTS 版本            |
| 用户权限     | 使用 root 用户 或 sudo 权限用户       | 所有设置都需要管理员权限              |
| 主机名唯一   | `hostnamectl set-hostname node1`      | 否则可能无法区分节点                   |
| hosts 配置   | 每个节点要能解析其他节点名            | 修改 `/etc/hosts` 加入节点映射         |
| 时间同步     | 安装 `chrony` 或 `ntp` 保证时间一致   | 时间不一致会导致证书验证失败          |

##### 二、关闭或配置系统功能（重要！）
| 项目                       | 命令                                                  | 原因                                                  |
|----------------------------|-------------------------------------------------------|-------------------------------------------------------|
| 关闭 swap                  | `swapoff -a` 并注释 `/etc/fstab` 中 swap 行         | Kubernetes 禁止使用 swap                              |
| 关闭防火墙（开发环境）     | `systemctl stop firewalld/ufw`                        | 节点间通讯可能被拦截，生产环境要配置策略              |
| 关闭 SELinux（CentOS）     | `setenforce 0` 并编辑 `/etc/selinux/config`          | 否则 kubelet 启动失败                                 |
| 配置 iptables              | `modprobe br_netfilter` + 设置 sysctl 参数           | 否则网络插件会出问题                                  |

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```
##### 三、容器运行时选择
因为k8s在1.24版本弃用了docker的支持 所以更支持用containerd来跑容器
| 选项         | 说明                                                 |
|--------------|------------------------------------------------------|
| containerd ✅ | 官方推荐，轻量稳定                                   |
| Docker ❌     | 已不推荐，需额外安装 cri-dockerd                    |

推荐安装 containerd：
```bash
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```
##### 四、安装 kubeadm / kubelet / kubectl（核心三件套）
添加 K8s 官方源 + 安装：
``` bash
sudo apt update && sudo apt install -y apt-transport-https curl

curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### 安装 k8s
这次我们采用的单节点



### crictl 和 nerdctl
你在 crictl 中能看到 nginx:latest 等镜像
但在 nerdctl images 里是空的
说明你的镜像目前**只存在于 containerd 的默认 namespace（k8s.io）**中，而 nerdctl 默认使用的是 moby namespace。
✅ 解决方案：切换 nerdctl 的 namespace
使用以下命令查看 containerd 中有哪些命名空间：
```bash
ctr namespaces list
```
你大概率会看到像这样的一些 namespace：
```bash
NAME     LABELS
default
k8s.io
moby
```
➤ 查看 k8s.io 中的镜像：
```bash
nerdctl --namespace k8s.io images
```
✅ 如果你想导出这些镜像
例如导出 nginx：
```bash
nerdctl --namespace k8s.io save nginx:latest -o /tmp/nginx.tar
```
然后在 moby 或其他你打算用来构建的 namespace 中导入：
```bash
nerdctl --namespace moby load -i /tmp/nginx.tar
```
📌 你可以设置默认 namespace（可选）
如果你想让 nerdctl 默认也用 k8s.io 的镜像，可以设置：
```bash
export CONTAINERD_NAMESPACE=k8s.io
```
加进 .bashrc 或 .zshrc 里就能永久生效。
操作目的
命令示例
查看镜像在哪些 namespace 下
ctr namespaces list
nerdctl 查看 k8s 镜像
nerdctl --namespace k8s.io images
导出 nginx 镜像
nerdctl --namespace k8s.io save nginx -o nginx.tar
导入镜像到默认环境
nerdctl load -i nginx.tar

nerdctl --namespace k8s.io images --digests

nerdctl --namespace k8s.io image prune




## 安装K8s nimikube 学习测试版本
``` bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
# x86架构 是amd64  如果是arm架构的需要换成 arm64
minikube start --driver=docker # 如果到这步卡住 是因为网络问题拉不到镜像 
# 可以换下面的方法
minikube start \
  --image-mirror-country=cn \
  --registry-mirror=https://registry.cn-hangzhou.aliyuncs.com \
  --driver=docker
# 如果还不可以 那我们就用docker去拉镜像 然后在minikube的启动参数上添加 --base-image=镜像名
# --base-image=registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.46
```
不知道自己是什么CPU架构？
``` bash
uname -m
# aarch64   aarch64就是arm架构
```
## K8s命令
### 🧭 集群与上下文管理
| 操作             | 命令 |
|------------------|------|
| 查看集群信息     | `kubectl cluster-info` |
| 查看当前上下文   | `kubectl config current-context` |
| 查看所有上下文   | `kubectl config get-contexts` |
| 切换上下文       | `kubectl config use-context <context-name>` |
| 设置默认命名空间 | `kubectl config set-context --current --namespace=<namespace>` |

### 📦 资源管理（Pod、Deployment、Service）

| 操作             | 命令 |
|------------------|------|
| 查看所有资源     | `kubectl get all` |
| 查看所有 Pod     | `kubectl get pods [-n <namespace>] [-o wide]` |
| 查看 Deployment  | `kubectl get deployments` |
| 查看 Service     | `kubectl get svc` |
| 创建资源         | `kubectl apply -f <file.yaml>` |
| 删除资源         | `kubectl delete -f <file.yaml>` |
| 删除 Pod         | `kubectl delete pod <pod-name>` |
| 滚动重启 Deployment | `kubectl rollout restart deployment <deployment-name>` |
| 修改资源配置     | `kubectl edit deployment <deployment-name>` |

### 🔍 查看与调试

| 操作             | 命令 |
|------------------|------|
| 查看日志         | `kubectl logs <pod-name>` |
| 实时查看日志     | `kubectl logs -f <pod-name>` |
| 查看容器日志     | `kubectl logs <pod-name> -c <container-name>` |
| 进入容器终端     | `kubectl exec -it <pod-name> -- /bin/bash` |
| 查看 Pod 信息    | `kubectl describe pod <pod-name>` |
| 查看集群事件     | `kubectl get events` |

### 🧱 命名空间管理

| 操作             | 命令 |
|------------------|------|
| 查看命名空间     | `kubectl get namespaces` |
| 创建命名空间     | `kubectl create namespace <name>` |
| 删除命名空间     | `kubectl delete namespace <name>` |

### 💾 存储管理（PVC、PV）

| 操作         | 命令 |
|--------------|------|
| 查看 PVC     | `kubectl get pvc` |
| 查看 PV      | `kubectl get pv` |

### 🛠 配置管理（ConfigMap、Secret）

| 操作             | 命令 |
|------------------|------|
| 创建 ConfigMap   | `kubectl create configmap <name> --from-literal=key=value` |
| 查看 ConfigMap   | `kubectl get configmap` |
| 创建 Secret      | `kubectl create secret generic <name> --from-literal=key=value` |
| 查看 Secret      | `kubectl get secrets` |

### 📈 网络与服务（Service、Ingress）

| 操作              | 命令 |
|-------------------|------|
| 创建 Deployment   | `kubectl create deployment <name> --image=<image>` |
| 创建 Service      | `kubectl expose deployment <name> --port=80 --target-port=8080 --type=NodePort` |
| 查看 Ingress      | `kubectl get ingress` |

### 🧪 临时调试与导出
| 操作             | 命令 |
|------------------|------|
| 临时启动容器     | `kubectl run -it --rm debug --image=busybox -- /bin/sh` |
| 导出资源为 YAML | `kubectl get deployment <name> -o yaml > deployment.yaml` |

