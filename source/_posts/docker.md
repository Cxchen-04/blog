---
title: docker
date: 2025-03-25 11:57:28
tags: [服务端,docker]
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
### 解决国内Docker镜像问题
由于2024年6月开始国内的大量docker镜像停服😤，Docker无法下载安装😭
此仓库致力于解决国内网络原因无法使用Docker的问题
使用Github Action将官网的安装脚本/安装包定时下载到本项目Release，供国内使用
官方安装包，安全可靠
每天自动定时同步，保证最新
作者：[技术爬爬虾](https://github.com/tech-shrimp/me)
B站，抖音，Youtube全网同名，转载请注明作者
``` Bash Linux
sudo curl -fsSL https://get.docker.com| bash -s docker --mirror Aliyun 
#一键安装命令

sudo curl -fsSL https://github.com/tech-shrimp/docker_installer/releases/download/latest/linux.sh| bash -s docker --mirror Aliyun
# 备用命令（每天自动从官网定时同步）

sudo curl -fsSL https://gitee.com/tech-shrimp/docker_installer/releases/download/latest/linux.sh| bash -s docker --mirror Aliyun
# 备用2 （如果github访问不了，可以使用gitee的链接）

sudo service docker start #启动Docker
```

### Centos安装docker
CentOS 7或更高版本
必须启用CentOS Extras存储库。默认情况下，此存储库已启用，但如果已禁用，则需要 重新启用它。
建议使用overlay2存储驱动程序。
如果以前安装的老版本（Docker名称是docker或docker-engine）请先删除
``` Bash centOS
sudo yum remove docker \
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
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
<!--                                                      -->
2.使用仓库安装(推荐)
2.1设置docker仓库
``` Bash centOS
sudo yum install -y yum-utils \
device-mapper-persistent-data
lvm2
sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```
2.2安装docker CE
``` Bash centOS
sudo yum install docker-ce docker-ce-cli container.io
 # 安装最新的Docker CE版本
yum list docker-ce --showduplicates | sort -r
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> container.io
#说明：<VERSION_STRING>，取上图中第二列中的第一个：或第一个数字到-之间的字符串，如18.09.6、18.06.2.ce等。
```
<!--                                                         -->
3.使用RPM程序包安装（适用于没有互联网接入的情况）
3.1下载所需要的Docker版本
下载地址： https://download.docker.com/linux/centos/7/x86_64/stable/Packages/
``` Bash centOS
sudo yum install /path/to/package.rpm
# 其中/path/to/package.rpm,为你下载下来的rpm包所在位置和文件名称
sudo systemctl start docker
# 启动Docker
sudo docker run hello-world
# 验证安装是否正确
```
更多信息: [Centos安装docker](https://www.coonote.com/docker/centos-install-docker.html)




### Ubuntu安装docker
Docker 需要在64位版本的Ubuntu上安装。此外，你还需要保证你的 Ubuntu 内核的最小版本不低于 3.10，其中3.10 小版本和更新维护版也是可以使用的。

``` Bash Ubuntu
uname -r 
3.11.0-15-generic
```

``` Bash Ubuntu
which wget # 查看你是否安装了wget

sudo apt-get update 
sudo apt-get install wget 
# 如果wget没有安装，先升级包管理器，然后再安装它。

wget -qO- https://get.docker.com/ | sh  
# 获取最新版本的 Docker 安装包

sudo docker run hello-world  # 验证 Docker 是否被正确的安装
 # 上边的命令会下载一个测试镜像，并在容器内运行这个镜像
```
更多信息: [Ubuntu安装docker](https://www.coonote.com/docker/ubuntu-install-docker.html)




### Macos安装docker
Homebrew 的 Cask 已经支持 Docker for Mac
``` Bash Macos
brew cask install docker
```
在载入 Docker app 后，点击 Next，可能会询问你的 macOS 登陆密码，你输入即可。之后会弹出一个 Docker 运行的提示窗口，状态栏上也有有个小鲸鱼的图标（）。

更多信息: [Macosa安装docker](https://www.coonote.com/docker/macos-intall-docker.html)




## 使用docker
### Pull镜像
#### 方案一 转存到阿里云
使用github Action将国外的Docker镜像转存到阿里云私有仓库，供国内服务器使用，免费易用
支持DockerHub, gcr.io, k8s.io, ghcr.io等任意仓库
支持最大40GB的大型镜像
使用阿里云的官方线路，速度快
项目地址: [Github](https://github.com/tech-shrimp/docker_image_pusher)

#### 方案二 镜像站
现在只有很少的国内镜像站存活
不保证镜像齐全,且用且珍惜
以下三个镜像站背靠较大的开源项目，优先推荐
| 项目名称   | 项目地址                                      | 加速地址                          |
| :--------- | :------------------------------------------- | :------------------------------- |
| 1Panel     | https://github.com/1Panel-dev/1Panel/        | https://docker.1panel.live       |
| Daocloud   | https://github.com/DaoCloud/public-image-mirror | https://docker.m.daocloud.io     |
| 耗子面板   | https://github.com/TheTNB/panel              | https://hub.rat.dev              |

linux配置镜像站
``` bash Linux
sudo vi /etc/docker/daemon.json #创建/打开daemon文件

{
    "registry-mirrors": [
        "https://docker.m.daocloud.io",
        "https://docker.1panel.live",
        "https://hub.rat.dev"
    ]
}
# 输入以上内容，vi按esc，输入:wq保存  nano输入ctrl+x保存退出

sudo service docker restart # 重启docker
```

#### 方案三 离线镜像
使用Github Action下载docker离线镜像 https://github.com/wukongdaily/DockerTarBuilder
#### 方案四 使用一键脚本
``` bash docker
bash -c "$(curl -sSLf https://xy.ggbond.org/xy/docker_pull.sh)" -s 完整镜像名
```
#### 方案五 使用Cloudflare worker 自建镜像加速
https://github.com/cmliu/CF-Workers-docker.io
#### 去哪里找镜像
https://docker.fxxk.dedyn.io/

更多信息: [Github Action](https://github.com/tech-shrimp/docker_installer?tab=readme-ov-file)

### 常用的操作命令

## Docker Dockerfile
### ✅ 一、什么是 Dockerfile？
Dockerfile 是一套“食谱”，告诉 Docker 如何构建你的镜像。
通过它，你可以：
	•	指定用哪个基础镜像（比如 Python、Node、Nginx）
	•	安装依赖
	•	拷贝文件
	•	执行命令
	•	设置运行服务时的默认命令
### ✅ 二、Dockerfile 语法结构（超清晰版）
``` Dockerfile
# 1. 指定基础镜像（必须有）
FROM 镜像名[:tag]

# 2. 设置容器中的工作目录（可选）
WORKDIR /app

# 3. 拷贝文件到镜像中
COPY ..

# 4. 安装依赖（可多条RUN）
RUN 命令

# 5. 设置环境变量（可选）
ENV 变量名=值

# 6. 设置容器启动时执行的命令
CMD ["命令","参数"]

# 7. 设置暴露的端口（可选，用于文档提升）
EXPOSE 端口号
```
### ✅ 三、最常见的语句详解 + 示例
| 指令     | 示例                          | 说明                             |
|----------|-------------------------------|----------------------------------|
| FROM     | FROM node:18                  | 基础镜像                         |
| WORKDIR  | WORKDIR /app                  | 切换目录（类似 cd）              |
| COPY     | COPY . .                      | 将当前目录复制到容器里           |
| RUN      | RUN npm install              | 构建期间执行命令                 |
| CMD      | CMD ["npm", "start"]         | 容器启动时运行（只能有一条）     |
| EXPOSE   | EXPOSE 3000                   | 说明服务监听的端口（仅文档提示） |
### ✅ 四、实战示例：Flask Web App
📁 项目结构：
``` bash
my-flask-app/
├── app.py
├── requirements.txt
└── Dockerfile
```
📄 app.py：
``` python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello,Docker!"

if __name__ == "__main__":
    app.run(host="0.0.0.0",port=5000)
```
📄 requirements.txt：
``` bash
flask
```
📄 Dockerfile：
``` Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY ..
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python","app.py"]
```
🛠 构建镜像并运行：
``` bash
docker build -t my-flask-app .
docker run -d -p 5000:5000 my-flask-app
```
浏览器访问：http://localhost:5000
🎉 成功！
### ✅ 五、常见问题与技巧
| 问题           | 原因与解决方式                                               |
|----------------|--------------------------------------------------------------|
| 容器运行后马上退出 | CMD 写错或服务没启动                                         |
| 镜像太大         | 换用 `-slim`、`-alpine` 镜像                                 |
| 文件没拷进去     | COPY 路径写错、`.dockerignore` 把它排除了                    |
| 构建慢           | 用 `.dockerignore` 排除不必要的文件（比如 `node_modules`） |

### ✅ 六、Dockerfile 小技巧合集
``` Dockerfile
# 编译阶段
FROM node:18 as build
WORKDIR /app
COPY ..
RUN nom install && npm run build

# 部署阶段（nginx）
FROM nginx
COPY --from=build /app/dist /usr/share/nginx/html

#设置环境变量
ENV PORT=8080
```
### ✅ 最后送你一句口诀 🧠
<b>FROM 定基础，WORKDIR 定位置，COPY 拷代码，RUN 装环境，CMD 启动它。</b>



## Docker Compose
### ✅ 一、Docker Compose是什么？
他是一个docker-compose.yml文件：
	•	描述多个服务（比如 web、db、redis）
	•	指定端口、挂载、网络、环境变量等
	•	一条命令就能启动：docker-compose up
### ✅ 二、基本语法结构
```Yaml
version: '3'    #Compose文件版本,一般用‘3’或‘3.8’

services:       #定义所有容器服务
    服务名1:
        image: 镜像名 或 buld路径
        ports:
            - "本地端口：容器端口"
        volumes:
            - "本地路径：容器路径"
        environment:
            - "变量名=值"
        depends_on:
            - 其他服务名（启动顺序）

    服务名2:
        ....
```
### ✅ 三、最简单示例：部署一个带自定义挂载的 Nginx
```Yaml
my-compose-project/
├── docker-compose.yml
├── html/
│   └── index.html
└── nginx.conf
```
📄 docker-compose.yml 内容：
```Yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
```
📌说明：
	•	端口映射：8080 → 容器内80
	•	挂载本地 ./html 到 nginx 的网页根目录
	•	挂载本地配置文件替换默认 nginx 配置
✅ 启动：
```bash Ubuntu
docker-compose up -d # 或者 docker compose up -d
```
✅ 停止：
```bash Ubuntu
docker-compose down -v # 或者 docker compose down -v
```
### ✅ 四、多服务组合示例（Node.js + MongoDB）
```Yaml
version: '3'

services:
    app:
        build: .
        ports:
            - "3000:3000"
        environment:
            - MONGO_URL=mongodb://mongo:27017/mydb
        depends_on:
            - mongo
    
    mongo:
        image: mongo
        ports:
            - "27017:27017"
```
📌说明：
	•	app 服务用 Dockerfile 构建
	•	依赖 Mongo 容器，并通过服务名 mongo 进行连接
### ✅ 五、常见字段说明（适合记住）
| 字段           | 用法                                              |
|----------------|---------------------------------------------------|
| image:         | 使用已有镜像                                      |
| build:         | 用 Dockerfile 构建镜像                            |
| ports:         | 本地端口:容器端口                                 |
| volumes:       | 本地路径:容器路径                                 |
| environment:   | 设置环境变量                                      |
| depends_on:    | 设置服务启动顺序                                  |
| restart:       | 容器崩溃时是否自动重启（如 always、on-failure）  |
| networks:      | 设置自定义网络（可多个服务通信）                  |

### ✅ 六、实用命令大全
```bash Ubuntu
# 启动服务
docker-compose up -d # 或者 docker compose up -d

# 停止并删除容器
docker-compose down # 或者 docker compose down

# 查看运行日志
docker-compose logs # 或者 docker compose logs

# 查看服务状态
docker-compose ps # 或者 docker compose ps

# 进入容器
docker-compose exec 服务名 bash # 或者 docker compose exec 服务名 bash
```


