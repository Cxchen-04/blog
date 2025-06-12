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

### k8s解决了什么问题？
你用 Docker 启动了一个 Web 服务：
```bash
docker run -d -p 80:80 my-app
```
很好用没错。但是后来你发现问题越来越多；
	1.	要部署 10 个容器怎么办？怎么管理？
	2.	容器挂了怎么办？手动重启太麻烦。
	3.	多个服务器如何分配资源？怎么调度？
	4.	配置、密钥、服务发现怎么统一管理？
	5.	如何升级不影响用户？如何快速回滚？
🎯 这些痛点就是 Kubernetes 诞生的原因：它是为了解决**大规模容器管理、自动化调度和高可用性问题**而生的。
### 必须知道的核心概念

| 概念                    | 小白理解方式                                                                 |
|-------------------------|------------------------------------------------------------------------------|
| Pod                     | 是 K8s 中的最小 “部署单位”，通常一个 Pod 包含一个容器，就像一个房间住一个人    |
| Node                    | 运行 Pod 的主机，可以是真实机器或虚拟机                                     |
| Cluster                 | 多个 Node 组成一个 K8s 集群                                                  |
| Deployment              | 定义一组 Pod 怎么创建、升级、回滚，保持数量稳定                             |
| Service                 | 为一组 Pod 提供统一访问入口，类似 “前台服务台”                              |
| Namespace               | 相当于集群里的分区，用于资源隔离（比如测试环境和生产环境）                   |
| Ingress                 | 让外部可以通过域名访问集群内服务                                            |
| Volume / PVC            | 给容器挂载数据盘，让数据不会因容器消失而丢失                                 |
| ConfigMap / Secret      | 存配置文件和密码的地方，避免硬编码进镜像                                     |
| RBAC 权限控制           | 限制谁能做什么操作，防止权限过大出问题                                       |
| Kubelet / Controller / Scheduler | 是 K8s 的“大脑和神经系统”，自动完成任务调度和容器生命周期管理     |

### 必须知道的容器概念
| 概念               | 简要说明                                                                 |
|--------------------|--------------------------------------------------------------------------|
| 镜像 (Image)       | 是程序 + 环境的打包文件，相当于 App 安装包                              |
| 容器 (Container)   | 是运行中的镜像，是进程不是虚拟机                                        |
| Docker             | 是最常用的容器引擎，K8s 用它（或 containerd）运行容器                   |
| 容器编排           | 多个容器如何自动部署、扩容、重启，就是 K8s 做的事                        |
| YAML 配置          | K8s 所有资源都靠 YAML 定义，需要能看懂基本结构                           |

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


## 安装K8S 内网生产环境
### 准备工作
机器数量3个，采用1master+2node的架构部署集群
| IP              | 主机名       | 角色     | 数量 | 配置      |
|-----------------|--------------|----------|------|-----------|
| 192.168.6.235   | k8s-master    | master   | 1    | 16C 32G   |
| 192.168.7.75    | k8s-node1     | node     | 1    | 16C 32G   |
| 192.168.7.76    | k8s-node2     | node     | 1    | 16C 32G   |

### 部署 K8S
1. 更改主机名 配置yum源
按照上述，更改主机名
```bash
hostnamectl set-h^Ctname k8s-master
hostnamectl set-h^Ctname k8s-node1
hostnamectl set-h^Ctname k8s-node2
# 注意按照相应的版本调整阿里云的源地址
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo

```
2. 配置hosts文件
```ini
192.168.6.235 k8s-master
192.168.7.75 k8s-node1
192.168.7.76 k8s-node2
```
3. 关闭防火墙
```bash
# 临时关闭
sudo setenforce 0
# 永久禁用
sudo sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```
4. 禁用swap分区
```bash
# 临时关闭；关闭swap
sudo swapoff -a
# 永久关闭
sudo sed -i '/swap/d' /etc/fstab
# 可以通过这个命令查看swap是否关闭了
free
```
5. 安装必要模块

```bash
yum install  -y ipvsadm ipset sysstat conntrack libseccomp
# dummy0网卡和kube-ipvs0网卡：在安装k8s集群时，启用了ipvs的话，就会有这两个网卡。（将service的IP绑定在kube-ipvs0网卡上）
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
ipvs_modules="ip_vs ip_vs_lc ip_vs_wlc ip_vs_rr ip_vs_wrr ip_vs_lblc ip_vs_lblcr ip_vs_dh ip_vs_sh ip_vs_fo ip_vs_nq ip_vs_sed ip_vs_ftp nf_conntrack ip_tables ip_set xt_set ipt_set ipt_rpfilter ipt_REJECT ipip "
for kernel_module in \${ipvs_modules}; do
  /sbin/modinfo -F filename \${kernel_module} > /dev/null 2>&1
  if [ $? -eq 0 ]; then
    /sbin/modprobe \${kernel_module}
  fi
done
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules 
sh /etc/sysconfig/modules/ipvs.modules 
lsmod | grep ip_vs

# 转发 IPv4 并让 iptables 看到桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置所需的 sysctl 参数，参数在重新启动后保持不变
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 应用 sysctl 参数而不重新启动
sudo sysctl --system
# 检查是否安装正确
lsmod | grep br_netfilter
lsmod | grep overlay

# 确认 net.bridge.bridge-nf-call-iptables、net.bridge.bridge-nf-call-ip6tables 和 net.ipv4.ip_forward 系统变量在你的 sysctl 配置中被设置为 1
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```
6. 运行时，选择 containerd
```bash
# 可以直接安装
yum install -y containerd

# 下载安装包，然后解压即可
https://github.com/containerd/containerd/blob/main/docs/getting-started.md
```
7. 配置containerd，镜像加速，cgroup等
```bash
# 查看containerd版本
containerd --version

版本为 1.6.32

# 配置为aliyun的镜像源
mkdir /etc/containerd
containerd config default > /etc/containerd/config.toml
sed -i "s#registry.k8s.io/pause:3.8#registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.9#g"  /etc/containerd/config.toml

# 配置镜像加速
vim /etc/containerd/config.toml
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = ["https://docker-mirror.h3d.com.cn"]

# 配置cgroup
vim /etc/containerd/config.toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```
8. 配置k8s源

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
9. 安装kubelet、kubeadm、kubectl(所有节点都需要)

```bash
# 查看可用版本
yum list kubelet --showduplicates |grep 1.28

# 安装
yum install -y kubelet-1.28.2 kubeadm-1.28.2 kubectl-1.28.2 --disableexcludes=kubernetes
systemctl enable --now kubelet
```
10. 配置crictl(如果想要在所有节点操作，可以都安装)

```bash
# 如果每个节点都需要crictl，可以安装并配置
cat  <<EOF | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///var/run/containerd/containerd.sock
timeout: 5
image-endpoint: unix:///var/run/containerd/containerd.sock

EOF
```
11. 初始化(仅在master节点操作)

```bash
kubeadm init --kubernetes-version=1.28.15 \
--apiserver-advertise-address=193.168.6.235  \
--image-repository  registry.aliyuncs.com/google_containers \
--pod-network-cidr=10.1.0.0/16
--service-cidr=10.96.0.0/12
```
输出如下
```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.6.235:6443 --token r10sv8.8wdmesq0hl16ztxy \
    --discovery-token-ca-cert-hash sha256:fdcbd538e705dd0d68ea7d0aea289f09d1b3e461bab539c2f0b4262ae36762b9 
```
12. 按照初始化输出，进行操作，并添加master污点

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf

# 添加污点，避免业务进程调度到master节点
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```
13. 检查集群状态

```bash
kubectl get nodes

kubectl get pods -A
```
14. 安装网络插件

```bash
kubectl apply -f scripts/calico.yaml
```

15. 检查集群并添加worker节点

```bash
# 所有worker节点上执行
kubeadm join 192.168.6.235:6443 --token r10sv8.8wdmesq0hl16ztxy \
    --discovery-token-ca-cert-hash sha256:fdcbd538e705dd0d68ea7d0aea289f09d1b3e461bab539c2f0b4262ae36762b9 

# 查看node状态
kubectl get nodes
```





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
kubectl get ns local-path-storage -o json | jq 'del(.spec.finalizers)'| kubectl replace --raw "/api/v1/namespaces/local-path-storage/finalize" -f -

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


## Helm K8s的「包管理工具」
### 什么是 Chart？它有什么作用？
Chart 是 Helm 的核心概念，简单说就是：

Chart 是一个打包好的 Kubernetes 应用模板，包含了一组 YAML 文件 + 参数配置文件。

一个 Chart 包含的内容通常如下：
```bash
mychart/
├── Chart.yaml           # Chart 的元信息（名称、版本等）
├── values.yaml          # 默认参数（可被 -f 覆盖）
├── templates/           # 存放实际的 YAML 模板
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl     # 公共模板函数
```
Chart 的作用：

| 作用       | 说明                                                                 |
|------------|----------------------------------------------------------------------|
| 📦 打包       | 把一套复杂的部署逻辑（Deployment、Service、Ingress 等）统一封装     |
| 🔁 复用       | 你可以反复在不同集群、不同环境中使用它，只需要传不同参数              |
| 🧩 参数化     | 通过 `values.yaml` 配置文件，灵活控制副本数、镜像、端口等             |
| 🔍 版本管理   | Chart 支持版本控制，与 Helm release 一起回滚升级                     |
| 🚀 快速部署   | 一条命令即可部署复杂应用（例如 Loki、Prometheus、GitLab 等）         |


### helm命令
##### 📦 Chart 仓库管理
```bash
helm repo add <name> <url>             # 添加 Helm 仓库
helm repo list                         # 查看已添加的仓库
helm repo update                       # 更新仓库信息
helm search repo <keyword>             # 在仓库中搜索 chart
```
##### 📥 安装 / 升级
```bash
helm install <release-name> <chart> -n <namespace> -f <values.yaml>
# 安装一个 chart，比如：
# helm install loki grafana/loki-stack -n monitor -f loki-values.yaml

helm upgrade <release-name> <chart> -n <namespace> -f <values.yaml>
# 升级已有的 release，但不会自动安装

helm upgrade --install <release-name> <chart> ...
# 安装或升级（推荐用于脚本或 CI/CD）

helm uninstall <release-name> -n <namespace>
# 卸载 release

helm rollback <release-name> <revision> -n <namespace>
# 回滚到历史版本
```
##### 🔍 状态 / 日志 / 调试
```bash
helm list -n <namespace>             # 查看某个命名空间下的 release
helm status <release-name> -n <namespace>  # 查看 release 状态
helm get values <release-name> -n <namespace>       # 查看已部署时使用的 values
helm get manifest <release-name> -n <namespace>     # 查看渲染后的 Kubernetes YAML
helm history <release-name> -n <namespace>          # 查看历史版本
```

## Kustomize 是什么
🧩 一句话总结 Helm 与 Kustomize 的区别：

Helm 是“打包部署模板”系统，Kustomize 是“YAML 衍生变种”系统。

📦 Kustomize 是什么？

Kustomize 是 Kubernetes 原生支持的配置管理工具（kubectl 从 1.14 起内置支持），它通过覆盖和组合 YAML 文件来实现“相同基础、不同环境”的部署配置。

你不用写模板语法（像 Helm 的 {{ }}），而是通过文件夹 + patch 的形式组合 YAML。
🧱 Kustomize 的结构（典型目录示例）：
```bash
my-app/
├── base/   # 基础部署逻辑，适用于所有环境
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/  # 环境专属配置，通过 patch 或变量覆盖基础配置
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   ├── prod/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
```
📌 Kustomize 支持的功能：

| 功能                          | 说明                                      |
|-------------------------------|-------------------------------------------|
| namePrefix / nameSuffix       | 给资源名加前缀 / 后缀，避免冲突           |
| configMapGenerator / secretGenerator | 自动生成 ConfigMap / Secret         |
| patchesStrategicMerge         | 部分字段打补丁                            |
| images 替换                   | 可以指定镜像版本                          |
| 资源合并 / 继承               | 支持多层嵌套管理 base 和 overlay           |

🔍 Helm vs Kustomize：核心对比表

| 项目           | Helm                                           | Kustomize                                           |
|----------------|------------------------------------------------|-----------------------------------------------------|
| 模板机制       | Go 模板语法 `{{ }}`，灵活但复杂               | 原生 YAML + Patch，简单直观                         |
| 应用封装       | 支持 Chart 打包，版本控制                      | 无打包，仅结构化组织目录                            |
| 参数注入方式   | values.yaml                                     | patch.yaml + kustomization.yaml                     |
| 适合场景       | 第三方应用部署（如 Prometheus、Harbor）        | 自研项目多环境管理（如 dev/test/prod）              |
| 发布版本回滚   | Helm 有 Release 管理，支持版本和回滚           | 无 Release 概念，靠 Git 控制                        |
| 安装方式       | `helm install`                                 | `kubectl apply -k` 或 `kustomize build`             |


🧠 可以这样记：
Helm 更像是：yum / apt —— 安装别人做好的包（官方、第三方组件）
	•	Kustomize 更像是：自己写 docker-compose.override.yml —— 在自己 YAML 基础上做灵活的环境变体管理

🚀 举个实际例子
```bash
# 你在用 Helm 安装 Grafana：
helm install grafana grafana/grafana -f prod-values.yaml

# 你在用 Kustomize 部署自己开发的 app：
kubectl apply -k overlays/prod
```


## K8s进阶

① 调度相关
	•	NodeSelector / Affinity / Taints & Tolerations：精细控制 Pod 在哪些节点上运行。
	•	nodeSelector：简单标签匹配。
	•	affinity：支持多条件、亲和/反亲和调度。
	•	taints & tolerations：用于“拒绝非特定容忍条件的 Pod”调度。
	•	优先级与抢占（Priority & Preemption）：高优先级的 Pod 可以驱逐低优先级 Pod。

⸻

② 工作负载控制器
	•	StatefulSet：适用于有状态应用（如数据库），提供稳定的网络标识和持久存储。
	•	DaemonSet：确保每个节点都运行一个 Pod（常用于日志采集、监控等）。
	•	Job / CronJob：一次性任务或周期性任务，类似 Linux 的 cron。
	•	Horizontal Pod Autoscaler（HPA）：根据 CPU 或自定义指标自动伸缩 Pod 数量。
	•	Vertical Pod Autoscaler（VPA）：自动调整 Pod 的资源请求（目前在很多场景下仍谨慎使用）。
	•	Cluster Autoscaler：节点资源不足时自动扩容底层节点。

⸻

③ 网络与服务发现
	•	NetworkPolicy：K8s 的防火墙，可以限制 Pod 间通信。
	•	Service Mesh（如 Istio、Linkerd）：用于流量管理、服务观察、熔断、A/B 测试等高级服务治理。

⸻

④ 存储与数据管理
	•	StorageClass：定义不同的存储策略（慢速、高IO等），与 PVC 配合使用。
	•	VolumeSnapshot / VolumeClone：实现 PVC 快照和克隆（适合备份、还原）。
	•	CSI（Container Storage Interface）：插件化的存储驱动标准，支持云厂商和自定义存储。

⸻

⑤ 安全
	•	RBAC（Role-Based Access Control）：基于角色的权限控制。
	•	PodSecurityPolicy（已废弃）/ Pod Security Admission：控制 Pod 的安全行为（如是否允许特权模式）。
	•	NetworkPolicy：限制网络访问权限。
	•	ServiceAccount & Secrets：身份管理和加密敏感信息。

⸻

⑥ 可观测性与调试
	•	Events / Probes / Metrics API：诊断运行状况。
	•	日志采集：Loki / ELK / Fluentd / Promtail：结合 DaemonSet 收集集群日志。
	•	Tracing：使用 Jaeger/Zipkin 等进行分布式追踪。
	•	监控告警：Prometheus + Grafana + Alertmanager：完整的指标、告警、可视化体系。

⸻

⑦ 高级部署策略
	•	Canary / Blue-Green / RollingUpdate / Recreate：支持多种升级策略。
	•	Kustomize / Helm：更强大的资源打包和管理方式，适合复杂项目。

⸻

⑧ 平台扩展能力
	•	Operator（CRD + Controller）：自定义控制器管理非原生资源，比如部署自己的数据库、缓存。
	•	Webhooks（Mutating / Validating）：用来在资源创建前做拦截和修改。
	•	Admission Controller：控制资源创建和更新的“守门员”。
