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

## 安装K8s
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
1. 🚀 集群与配置
``` bash
# 查看集群信息
kubectl cluster-info

# 查看当前上下文
kubectl config current-context

# 设置上下文（切换集群、命名空间等）
kubectl config use-context <context-name>

# 查看所有上下文
kubectl config get-contexts
```
2. 📦 资源管理（Pods, Deployments, Services等）

``` bash
# 查看所有资源
kubectl get all

# 查看所有Pod
kubectl get pods [-n <namespace>] [-o wide]

# 查看Deployment
kubectl get deployments

# 查看Service
kubectl get svc

# 创建资源
kubectl apply -f <file.yaml>

# 删除资源
kubectl delete -f <file.yaml>
kubectl delete pod <pod-name>

# 更新资源
kubectl edit deployment <deployment-name>

# 滚动更新
kubectl rollout restart deployment <deployment-name>
```
3. 🔍 查看与调试
``` bash
# 查看 Pod 日志
kubectl logs <pod-name>
kubectl logs -f <pod-name>  # 实时查看

# 查看某个容器的日志（多容器 Pod）
kubectl logs <pod-name> -c <container-name>

# 进入 Pod（终端）
kubectl exec -it <pod-name> -- /bin/bash

# 描述资源
kubectl describe pod <pod-name>

# 查看事件
kubectl get events
```
4. 💾 存储（PVC, PV）
``` bash
# 查看持久卷声明
kubectl get pvc

# 查看持久卷
kubectl get pv
```
5. 🎯 部署与服务（Deployment, Service, Ingress）
``` bash
# 创建 Deployment
kubectl create deployment <name> --image=<image-name>

# 对外暴露端口（创建 Service）
kubectl expose deployment <deployment-name> --port=80 --target-port=8080 --type=NodePort

# 查看 Ingress
kubectl get ingress
```
6. 🔧 配置资源（ConfigMap, Secret）
``` bash
# 创建 ConfigMap
kubectl create configmap <name> --from-literal=key=value

# 查看 ConfigMap
kubectl get configmap

# 创建 Secret
kubectl create secret generic <name> --from-literal=password=123456

# 查看 Secret
kubectl get secrets
```
7. 🧪 调试和测试
``` bash
# 运行一个临时Pod进行测试
kubectl run -it --rm debug --image=busybox -- /bin/sh

# 查看Pod状态
kubectl get pod <pod-name> -o json
```
8. 📄 YAML 生成和导出
``` bash
# 导出资源为yaml
kubectl get deployment <name> -o yaml > deployment.yaml
```
9.  🚫 删除资源
``` bash
# 删除所有资源
kubectl delete all --all

# 删除命名空间
kubectl delete namespace <name>
```