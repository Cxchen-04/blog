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

### 使用 Jenkins
#### 下载 jenkis
💡 Jenkins 安装位置建议
| 安装位置              | 适合人群                   | 优点             |     缺点         |
|-------------------|--------------------------|----------------------------|------------|
| ✅ 本地开发机   | 开发者、测试者、博客站点维护者    | 安装简单，易调试，不涉及公网安全问题     |  电脑关机就不能自动部署了，不能远程持续运行  |
| ✅ 云服务器         | 想让 Jenkins 24小时在线自动部署 | 持久在线，适合多人协作    | 安装和配置略复杂，要注意安全（如开放 8080 端口、权限管理） |
#### Mac本地安装Jenkins（最简单方式）
1. 使用 Homebrew 安装 Jenkins
```bash
brew install jenkins-lts # 下载jenkins

brew services start jenkins-lts # 启动jenkins
```
默认 Jenkins 会跑在 http://localhost:8080 上，你可以直接用浏览器打开它。
2. 输入管理员密码（第一次登录）
页面标题是：Unlock Jenkins（解锁 Jenkins）
第一次打开 Jenkins 会让你输入密码，你可以运行
```bash
cat /Users/你的用户名/.jenkins/secrets/initialAdminPassword
# jenkins界面会有提示
```
复制输出的那串密码，粘贴回 Jenkins 页面里，点【继续】。
3. 安装插件
系统会问你：
	•	【安装推荐的插件】（Install suggested plugins）✅ 推荐选择这个
	•	【选择插件要安装】（Select plugins to install）
🔘 请点击第一个【安装推荐插件】，它会自动安装常用插件（比如 Git、SSH、Pipeline 等）。
💡 提示：这个过程需要几分钟时间。耐心等待插件安装完毕。
4. 创建第一个管理员用户
安装插件后，它会进入这个页面，让你创建一个账户：
	•	用户名（Username）: 你自己定义一个
	•	密码（Password）: 自己设
	•	完整名称（Full name）: 随意填写
	•	邮箱地址（Email address）: 推荐填真实邮箱
填好后点击【保存并继续（Save and Continue）】。
5. 配置 Jenkins URL
系统会提示你默认访问地址，例如：http://localhost:8080
保持默认，点击【保存并完成（Save and Finish）】。
#### 创建Pipeline项目
1. 我们来到jenkins的web主界面，点击左上角的 **Jenkins**，即可到达首页
  1. 点击【新建任务】（New Item）
  2. 输入任务名称，例如：deploy-hexo-blog
  3. 选择类型：【流水线（Pipeline）】✅
  4. 点击【确定】
2. 在 **Pipeline** 页面中配置
到最下面的 “流水线（Pipeline）” 区块，（推荐在vscode中编写好Jenkinsfile之后复制到这里）
	1.	定义方式：选择【Pipeline script】
	2.	在“Script”框中粘贴以下内容（我们先用最基础的版本）

```groovy
pipeline {
    agent any
    environment {
        DEPLOY_SERVER = '你的服务器ip'
        DEPLOY_USER = 'root'    // 你服务器的用户
        DEPLOY_PATH =  '/var/www/html'   // 你要上传到的目录
        SSH_CREDENTIALS_ID = 'ecs-ssh'   // 凭证
        LOCAL_HEXO_DIR = '/Users/macos/Desktop/blog'  // 本地的目录(绝对路径)
    }
    stages {
        stage('压缩public文件夹'){
            steps {
                sh """
                    cd ${env.LOCAL_HEXO_DIR}
                    zip -r ../public.zip ./public
                """
            }
        }
        stage('删除之前的public文件夹'){
            steps {
                script {
                    sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
                        sh """
                            ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && rm -rf public'
                        """
                    }
                }
            }
        }
        stage('上传并部署'){
            steps {
                script {
                    sshagent (credentials: ["${env.SSH_CREDENTIALS_ID}"]) {
                        sh """
                            scp ${env.LOCAL_HEXO_DIR}/../public.zip ${DEPLOY_USER}@${DEPLOY_SERVER}:${DEPLOY_PATH}/
                            ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && unzip -o public.zip && rm -f public.zip'
                            ssh ${DEPLOY_USER}@${DEPLOY_SERVER} 'cd ${DEPLOY_PATH} && chmod -R 755 public'
                        """
                    }
                }
            }
        }
    }
}
```
3. 点击页面底部的【保存】按钮
#### 运行部署任务！
1.	回到项目页面，点击左侧【立即构建（Build Now）】
2.	Jenkins 会开始执行：
•	压缩你本地的public文件夹
•	通过 SSH 登录到 ECS 删除之前的public文件夹
•	上传文件到你的部署目录
3.	点击左侧【构建历史】中的蓝色小球 → 查看【控制台输出（Console Output）】
✅ 如果看到 Finished: SUCCESS 就说明部署成功啦！
#### 设置凭证
刚才我们设置了 **SSH_CREDENTIALS_ID** 由于我们是刚刚创建好的jenkins还没有配置凭证
```ini
[ssh-agent] Could not find specified credentials: ecs-ssh
```
🔍 Jenkins 报错：找不到名为 ecs-ssh 的凭据
##### 添加 SSH 凭据并命名为 ecs-ssh
🔧 步骤如下：
	1.	打开 Jenkins 主界面
	2.	点击左侧【凭据（Credentials）】
	•	如果你找不到这个入口，也可以通过：
	•	【Manage Jenkins】→【Security】或【Manage Credentials】
	3.	选择：(global) 全局作用域
	4.	点击左边【Add Credentials】（添加凭据）
📝 添加 SSH 凭据填写内容如下：


### 使用 Github Actions
#### 目录结构
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
####  GitHub Actions自动部署脚本
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
####  配置服务器 SSH 密钥
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
#### 修改 REMOTE_USER、REMOTE_HOST、TARGET
比如你服务器用户名是 root，公网 IP 是 1.2.3.4，部署路径是 /var/www/html，那你就填
``` Yaml
REMOTE_USER: root
REMOTE_HOST: 1.2.3.4
TARGET: /var/www/html
```
#### 最后 推送试试！
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
