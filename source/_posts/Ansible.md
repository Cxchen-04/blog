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

3. Playbook（剧本）📜
你写的一份 .yaml 文件，定义了“在哪些机器上执行哪些任务”。
```yaml
- hosts: all
  become: yes
  tasks:
    - name: 安装nginx
      apt:
        name: nginx
        state: present
```
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

## 使用Ansible
1. 🧾 第一步：准备你的 Ansible 主机清单（Inventory）
创建一个hosts.ini的文件：
```ini
[all]
192.168.1.101
192.168.1.102
192.168.1.103
192.168.1.104
```
📌 说明：这里定义了一个叫 webservers 的组，包含3台要删除 Nginx 的服务器 IP。你可以换成你自己的服务器地址。
2. 🛠️ 第二步：写一个下载 Nginx 的 Playbook
创建一个名为 install_nginx.yml 的 YAML 文件：
```yaml
- name: install nginx  # 此任务名称
  hosts: all        # 这里的all指向的是hosts文件中的[all]
  become: yes       # 使用 sudo 权限执行
  tasks:      # 下面要执行的具体任务列表
    - name: install nginx    # 第一个任务 安装 nginx
      apt:      # 使用 apt 模块
        name: nginx     # 要安装的软件包名: nginx
        state: present    # 保证软件包“已安装”状态（不存在就会被安装）
        update_cache: yes     # 更新 apt 源缓存
    - name: start nginx      # 第二个任务 启动 nginx
      service:   # 使用 service模块操作系统服务
        name: nginx     # 操作的服务名称: nginx
        state: started    # 保证服务处于 “启动”状态
        enabled: yes      #设置为开机自启
```
3. ▶️ 第三步：运行这个 Playbook 命令
```bash
ansible-playbook -i ../hosts install_nginx.yml 
```
## 实战
### 批量安装 Kubernetes
1. 第一步 先创建inventory
```bash
mkdir -p ~/k8s
cd k8s
nano hosts.ini
```
编写hosts文件
```ini
[all]
172.17.50.246 ansible_user=h3d ansible_become=true ansible_become_method=sudo
172.17.50.247 ansible_user=h3d ansible_become=true ansible_become_method=sudo
172.17.50.248 ansible_user=h3d ansible_become=true ansible_become_method=sudo
172.17.50.249 ansible_user=h3d ansible_become=true ansible_become_method=sudo
[master]
172.17.50.246
[worker]
172.17.50.247
172.17.50.248
172.17.50.249
```
#### 设置基础环境
用ansible来批量设置master和worker的环境
1. 创建playbook
```bash
nano setup.yml
```
```yml
- name: base environment (shuting swap)
  hosts: all
  become: yes
  tasks:
    - name: shuting swap
      shell: swapoff -a
      ignore_errors: true
    - name: permenant shuting swap
      replace:
        path: /etc/fstab
        regexp: '^\s*(.+?\s+)+swap\s+'
        replace: '# \1'
      ignore_errors: true

    - name: enable kernel module
      shell: |
        modprobe overlay
        modprobe br_netfilter
      ignore_errors: true

    - name: deploy kernel arguments
      copy:
        dest: /etc/sysctl.d/k8s.conf
        content: |
          net.bridge.bridge-nf-call-ip6tables = 1
          net.bridge.bridge-nf-call-iptables = 1

    - name: apply sysctl configration
      shell: sysctl --system
```
#### 容器选择
因为k8s在1.24版本移除了docker的支持，因此我们推荐使用containerd来座位k8s的容器，具体为什么我们移步 __K8s__ 章节观看
1. 创建playbook
```bash
nano install-containerd.yml
```

```yml
- name: install containerd
  hosts: all
  become: yes
  tasks:
    - name: install containerd
      apt:
        name: containerd
        state: present
        update_cache: yes
    - name: make containerd configuration file
      file:
        path: /etc/containerd
        state: directory
    - name: create default configuration file
      shell: containerd config default > /etc/containerd/config.toml
      args:
        creates: /etc/contaierd/config.toml
    - name: enable SystemdCgroup (compatable kubernetes)
      replace:
        path: /etc/containerd/config.toml
        regexp: 'SystemdCgroup = false'
        replace: 'SystemdCgroup = true'
    - name: restart && enable containerd service
      systemd:
        name: containerd
        enabled: yes
        state: restarted
```
我们在主节点生成配置文件
```bash
containerd config default > /etc/containerd/config.toml
```
找到这个位置，确认存在并设置为阿里云镜像：
```bash
[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.8"
```
向下继续找到 runc 配置段，设置如下（一定要加上 SystemdCgroup = true）：
```bash
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```
#### 安装k8s
1. 创建playbook
```bash
nano install-k8s.yml
```
写入以下配置
```yml
- name: install kubernetes module
  hosts: all
  become: yes
  tasks:
    - name: add Kubernetes source
      copy:
        dest: /etc/apt/sources.list.d/kubernetes.list
        content: |
          deb [trusted=yes] https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
    - name: update apt source
      apt:
        update_cache: yes

    - name: install kueblet kubeadm kubectl
      apt:
        name:
          - kubelet
          - kubeadm
          - kubectl
        state: present
        update_cache: yes

    - name: lock version
      shell: apt-mark hold kubelet kubeadm kubectl
```
2. 运行
```bash
ansible-playbook -i ../hosts install-k8s.yml --ask-pass --ask-become-pass
```
