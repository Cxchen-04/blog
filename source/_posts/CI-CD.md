---
title: DevOps 世界的灵魂技术：CI/CD 流水线
date: 2025-04-13 12:21:48
cover: /images/cicd.png
---

DevOps 世界的灵魂技术：CI/CD 流水线

## 什么是 CI/CD 流水线
CI/CD = 持续集成 + 持续交付 / 持续部署
是一整套“自动化流水线”，用来让软件开发从提交代码 → 自动构建 → 自动测试 → 自动部署，一条龙完成
### CI/CD 是干嘛的
``` Plaintext
你写代码 → 推送到 GitHub / GitLab
           ↓
      ✅ CI 阶段（持续集成）
      - 自动编译打包
      - 自动跑单元测试
      - 自动代码检查
           ↓
      🚀 CD 阶段（持续交付 / 部署）
      - 自动构建 Docker 镜像
      - 自动发布到测试 / 生产环境
      - 自动通知（钉钉 / Slack）
```
### CI/CD 各阶段都干了啥？
| 阶段              | 中文名                   | 做什么举动？                            |
|-------------------|--------------------------|-----------------------------------------|
| CI（持续集成）     | Continuous Integration    | 自动测试 + 打包构建                     |
| CD（持续交付）     | Continuous Delivery       | 自动发布到测试环境，人工确认上线        |
| CD（持续部署）     | Continuous Deployment     | 自动发布到生产环境，无人工干预          |

<p>典型的流水线长这样</p>

``` Yaml
# Gitlab CI 示例 .gitlab-ci.yml
stages:
    - build
    - test
    - deploy

build:
    stage: build
    script:
        - npm install 
        - npm run build
    artifacts:
        path:
            - dist/

test:
    stage: test
    script:
        - npm test

deploy:
    stage: deploy
    script:
        - docker build -t my-app .
        - docker push myregistry.com/my-app
        - ssh deploy@server 'docker pull my-app && docker restart my-app'
```
### 🛠 常用 CI/CD 工具平台有哪些？
| 工具                  | 说明                                 | 推荐指数                           |
|-----------------------|--------------------------------------|------------------------------------|
| GitHub Actions        | GitHub 原生支持，简单易上手          | ⭐⭐⭐⭐⭐                            |
| GitLab CI/CD          | GitLab 自带 CI/CD，集成度高           | ⭐⭐⭐⭐                             |
| Jenkins               | 开源界老牌 CI/CD 工具                 | ⭐⭐⭐⭐（适合自定义复杂流程）         |
| Gitea + Drone         | 私有部署组合                          | ⭐⭐⭐                              |
| CircleCI / Travis CI | 国外流行，GitHub 项目多               | ⭐⭐⭐（限免费额度）                  |

<p>💡 现实例子：部署一个 Vue 项目 + 后端服务</p>

1. 开发人员提交代码到 GitHub
2. GitHub Actions 自动触发：
•	运行测试
•	执行构建（npm run build）
•	打包为 Docker 镜像
•	推送到 Docker Hub / 私有仓库
•	SSH 登录服务器远程部署 / 使用 K8s 滚动更新
•	通知钉钉 / 邮件群“部署完成”

🎉 全程不用你手点，自动完成
### CI/CD 的价值总结
| 好处         | 描述                                             |
|--------------|--------------------------------------------------|
| 🧹 降低人为错误 | 减少部署出错、环境不一致等问题                    |
| 🚀 加快迭代速度 | 提交代码后几分钟就能上线                         |
| 📦 提高质量     | 每次提交都跑测试，代码更稳定                      |
| 🤝 提升团队协作 | 所有人按统一标准部署，不容易混乱                  |
| 🔄 易于回滚     | 版本打包有记录，可随时切回                       |

## 使用
以此博客作为实战项目。使用GitHub Actions作为流水线工具🔧
1. 目标是什么？
实现一件事：我改完博客内容，push 到 GitHub，服务器自动更新上线，不用我点压缩、FTP！
2. 目前的工具组合
| 工具                        | 用法                           | 说明                       |
|-----------------------------|--------------------------------|----------------------------|
| GitHub + GitHub Actions     | 托管代码 + 执行 CI/CD 流水线   | 免费、好用                 |
| SSH + rsync                 | 自动远程上传代码               | 替代 FTP，更快更稳         |
| Nginx                       | 你已经部署好了，用作 Web 服务  | 根目录挂载网站内容         |

### 目录结构
如果没有workflows 需要手动创建！
``` bash
我的博客/
├── .github/
│   └── workflows/
│       └── deploy.yml      👈 GitHub Actions 流水线脚本
│   └── dependabot.yml      
├── public/                 👈 构建后输出目录
├── package.json / config.yml
```
### GitHub Actions自动部署脚本
``` Yaml
name: 自动部署博客到服务器

on: 
   push:
        branches:
            - main  # 使用的分支，如果是master 修改这里

jobs:
    deploy:
        runs-on: ubuntu-latest

        steps:
            - name: 拉代码
              uses: actions/checkout@v3

            - name: 安装Node.js 
              uses: actions/setup-node@v3
              with:
                node-version: 18       
            
            - name: 安装依赖 & 构建博客 
              run: |
                npm install 
                npm install -g hexo-cli
                npx hexo generate
            # 如果只需要上传资源到服务器 可以省去安装node 和hexo的依赖

            - name: 上传到服务器
              uses: easingthemes/ssh-deploy@v2.1.5
              with:
                SSH_PRIVATE_KEY: ${{ secrets.SERVER_SSH_KEY }}
                REMOTE_USER: root   # 你的服务器用户名
                REMOTE_HOST: x.x.x.x    # 你的服务器IP
                TARGET: /var/www/html   # 你nginx的根目录 写到根上一个文件夹
                SOURCE: public/         # 你网页的资源
                
```
### 配置服务器 SSH 密钥
1. 在本地执行命令生成密钥：
``` bash
ssh-keygen -t rsa -b 4096 -C "you@example.com"
```
2. 拷贝公钥到服务器：
``` bash
ssh-copy-id your_user@your.server.ip
```
或者手动把 ～/.ssh/id_rsa.pub的内容追加到服务器的：
``` bash
~/.ssh/authorized_keys
```
3. 把私钥~/.ssh/id_rsa 的内容复制，粘贴到 GitHub 仓库中：
GitHub → Settings → Secrets → Actions → New repository secret
名字叫：SERVER_SSH_KEY
值是你私钥的内容（注意安全，不要泄露）

### 修改 REMOTE_USER、REMOTE_HOST、TARGET
比如你服务器用户名是 root，公网 IP 是 1.2.3.4，部署路径是 /var/www/html，那你就填
``` Yaml
REMOTE_USER: root
REMOTE_HOST: 1.2.3.4
TARGET: /var/www/html
```

### 最后 推送试试！
``` bash
git add .
git commit -m "首次添加自动部署"
git push origin main
```
🌟 然后进入 GitHub → Actions，就能看到它自动执行！

部署成功后，你就可以访问 http:// your.server.ip 查看效果！
💡 小建议：可选加一行日志打印
``` Yaml
- name: 查看构建结果文件夹
  run: ls -al public/ 
```