---
# layout: port
title: Ansible
date: 2025-04-23 20:08:44
tags: [运维,Ansible]
categories: [运维, Ansible]
cover: /images/ansible.png
---
Ansible 是一个自动化运维工具，可以帮你用“一段脚本”自动做一堆服务器上的重复工作，比如：装软件、改配置、重启服务、部署项目……
你可以把它理解成：
“我把要做的事情写进一本‘剧本’，Ansible 来帮我远程在服务器上一步步执行。
## 简介
🔑 Ansible 的工作机制（一句话总结）：你写好要做的事情（Playbook） → Ansible 用 SSH 连接到你的服务器 → 按步骤一条条执行任务。
🧱 Ansible 的基础组成部分：
1. Inventory（清单）🗂️
告诉 Ansible 要连接哪些服务器，用什么方式连接。
```ini
[webservers]
192.168.1.10
192.168.1.11

[db]
127.0.0.1 ansible_connection=local
```
2. Modules（模块）🧩
 Ansible 提供的功能小工具，比如“安装软件”、“创建文件”、“启动服务”等等，都是模块完成的。

| 模块                            | 用途                       |
|--------------------------|-----------|
| apt/yum             | 查看网络             |
| copy/template        | 创建网络            |
| service               | 查看网络详情       |
| user                 | 将容器加入某个网络   |
| command/shell       | 启动时指定网络        |

<!-- 3. Playbook（剧本）📜
你写的一份 .yaml 文件，定义了“在哪些机器上执行哪些任务”。
```yaml
- hosts: all
  become: yes
  tasks:
    - name: 安装nginx
      apt:
        name: nginx
        state: present
``` -->
4. Tasks(任务)🧾
Playbook 中的每一个动作，比如“安装 nginx”、“复制文件”，都叫一个任务（task）。
5. Variables（变量）📦
你可以给一些值起名字，方便复用。比如：
``` yaml
vars:
  nginx_port:8080
```
6. Roles（角色）📁
让你的项目结构更清晰，比如专门搞一个 “nginx” 的文件夹，里面是安装、配置、重启 nginx 的所有内容。

### 💡 Ansible 跟 Shell 脚本的区别？

| 项目       | Shell 脚本                | Ansible                                        |
|------------|---------------------------|------------------------------------------------|
| 写法       | Bash 命令串               | YAML 写的任务清单                              |
| 可读性     | 一般                      | 很高，易读                                     |
| 复用性     | 差                        | 好，支持变量和模块                             |
| 错误处理   | 要手写判断                 | 自动判断执行结果                               |
| 多机部署   | 要循环 + 远程命令          | 天生多机执行（Inventory 配置即可）             |