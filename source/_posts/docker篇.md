---
title: docker
date: 2025-03-25 11:57:28
tags: [服务端，docker]
categories: [服务端, docker]
cover: /images/docker.webp
---
Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从Apache2.0协议开源。
Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

## 简介

容器技术不仅限于docker，但是docker目前最为流行，docker容器技术的核心之一在于镜像文件。
镜像文件，通俗的理解就是一个进程运行时依赖的软件文件的集装箱。

  应用集群部署时，每台机器首先会拉取指定版本的镜像文件。安装镜像后产生了docker容器。由于所有机器的镜像文件一样，容器的软件版本故而一样。即使开发或运维中途修改了容器的软件版本，但是容器销毁时，软件的改动会随容器的销毁一起湮灭。当应用用已有的镜像文件重新部署时，生成的docker容器跟修改之前的容器完全一样。这也是Infrastructure as code思想带来的好处。

  容器如果要升级软件版本，那就修改镜像文件。这样部署时集群内所有的机器重新拉取新的镜像，软件因此跟着一起升级。软件版本混乱的问题，到docker这里，也就得到了完美的解决。

一个疑问：有了容器技术，生产环境为何还需要部署虚拟机？

虚拟机能做到硬件资源的彻底隔离，docker不行。虚拟机 和 docker各取长处，最佳CP。

## 安装docker

### Centos安装docker
CentOS 7或更高版本
必须启用CentOS Extras存储库。默认情况下，此存储库已启用，但如果已禁用，则需要 重新启用它。
建议使用overlay2存储驱动程序。
如果以前安装的老版本（Docker名称是docker或docker-engine）请先删除
``` Bash centOS
$ sudo yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine
```
上述操作只会删除docker本身，但老版本保存在/var/lib/docker/的内容，包括镜像、容器、卷和网络需要手动删除。

#### 安装方式
1.脚本安装(多用于测试和开发环境)

``` Bash centOS
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

2.使用仓库安装(推荐)
2.1设置docker仓库
``` Bash centOS
$ sudo yum install -y yum-utils \
device-mapper-persistent-data
lvm2
$ sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```
2.2安装docker CE
``` Bash centOS
$ sudo yum install docker-ce docker-ce-cli container.io
 # 安装最新的Docker CE版本
$ yum list docker-ce --showduplicates | sort -r
```
更多信息: [Centos安装docker](https://www.coonote.com/docker/centos-install-docker.html)


### Ubuntu安装docker
Docker 需要在64位版本的Ubuntu上安装。此外，你还需要保证你的 Ubuntu 内核的最小版本不低于 3.10，其中3.10 小版本和更新维护版也是可以使用的。

``` Bash centOS
$ uname -r 
3.11.0-15-generic
```

``` Bash Ubuntu
$ which wget # 查看你是否安装了wget

$ sudo apt-get update 
$ sudo apt-get install wget 
# 如果wget没有安装，先升级包管理器，然后再安装它。

$ wget -qO- https://get.docker.com/ | sh  
# 获取最新版本的 Docker 安装包

$ sudo docker run hello-world  # 验证 Docker 是否被正确的安装
 # 上边的命令会下载一个测试镜像，并在容器内运行这个镜像
```
更多信息: [Ubuntu安装docker](https://www.coonote.com/docker/ubuntu-install-docker.html)




### Macos安装docker
Homebrew 的 Cask 已经支持 Docker for Mac
``` Bash Macos
$ brew cask install docker
```
在载入 Docker app 后，点击 Next，可能会询问你的 macOS 登陆密码，你输入即可。之后会弹出一个 Docker 运行的提示窗口，状态栏上也有有个小鲸鱼的图标（）。

更多信息: [Macosa安装docker](https://www.coonote.com/docker/macos-intall-docker.html)




## 使用docker

``` bash 
$ hexo server
```

更多信息: [服务端](https://hexo.io/docs/server.html)

## 生成静态文件

``` macOS
$ hexo generate
```

更多信息: [生成](https://hexo.io/docs/generating.html)

## 部署到远程仓库

``` windows
$ hexo deploy
```

更多信息: [部署](https://hexo.io/docs/one-command-deployment.html)

