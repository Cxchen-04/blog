---
title: Python 在 Linux 运维中的应用
# For a new blog file keep date same as other posts to maintain style
# We'll use 2025-03-26 13:38:15 same as Python.md to maintain style.
date: 2025-03-26 13:38:15
tags: [运维, Python, 自动化]
categories: [运维, Python]
cover: /images/python.png
---

## Python 简介
Python 是一门简洁易读的编程语言，非常适合用来编写自动化脚本和运维工具。相比仅能执行简单命令的 Shell，Python 拥有更完善的语法、海量的第三方库以及跨平台能力，因此能更好地应对复杂场景。

- **易上手**：语法直观，入门门槛低。
- **生态丰富**：拥有成千上万的第三方库，可快速实现网络请求、数据处理、图形化等功能。
- **可扩展**：既能写几行脚本，也能构建大型应用。

下面的内容将介绍在 Linux 运维中常见的 Python 技术，帮助零基础的同学快速入门。

## 准备工作：安装与运行

大多数 Linux 发行版都自带 Python。可以通过 `python3 --version` 查看当前版本。
若需要手动安装，可在 Debian/Ubuntu 上执行：
```bash
sudo apt update && sudo apt install -y python3 python3-pip
```
运行 Python 脚本的方式有两种：
1. 直接调用解释器：`python3 your_script.py`
2. 在脚本开头添加 **shebang** 并赋予执行权限：
   ```bash
   #!/usr/bin/env python3
   print("Hello, world!")
   ```
   然后 `chmod +x your_script.py`，即可通过 `./your_script.py` 执行。

## 基础语法速览

```python
# 变量与数据类型
name = "Tom"
age = 18
is_admin = False

# 条件判断
if age >= 18:
    print("成年人")
else:
    print("未成年")

# 循环
for i in range(3):
    print(i)

# 函数
def greet(user):
    return f"Hello, {user}!"

print(greet(name))
```

虽然语法简单，但掌握变量、列表、字典、循环、函数等概念后，就能编写实用的小工具了。学习时可以边看边敲，效果更佳。

## 核心概念补充

- **缩进**：Python 使用缩进来划分代码块，通常为四个空格。
- **注释**：以 `#` 开头的是单行注释，多行注释可用三引号包裹。
- **变量作用域**：函数内部声明的变量仅在函数内可见。
- **脚本与模块**：`.py` 文件既可以直接执行，也可以被其他代码 `import`。
- **入口判断**：常用 `if __name__ == '__main__':` 在脚本独立运行时执行测试代码。

## 常用标准库
Python 自带的**标准库**提供了大量实用模块，在运维脚本中经常会用到：

- `os`：与系统交互，获取环境变量、遍历文件夹等。
- `sys`：与 Python 解释器自身相关的参数和函数。
- `subprocess`：执行系统命令并获取输出，适合替代 `os.system`。
- `shutil`：高级文件操作，如复制、压缩等。
- `pathlib`：面向对象的路径处理，更加直观易读。
- `logging`：标准化日志输出，便于排查问题。
- `argparse`：解析命令行参数，编写易用的 CLI 程序。
- `json`：读写 JSON 配置或接口数据。
- `re`：使用正则表达式进行文本匹配和替换。
- `socket`：编写底层网络通信脚本。

示例：使用 `subprocess` 执行 `ls` 命令并打印结果。
```python
import subprocess

res = subprocess.run(['ls', '-l', '/var/log'], capture_output=True, text=True)
print(res.stdout)
```

## 实用语法小贴士

- **列表推导式**：`[x * x for x in range(5)]` 让循环生成列表更简洁。
- **with 语句**：自动管理文件或网络连接的关闭，避免忘记释放资源。
- **格式化字符串**：使用 `f"当前用户: {name}"` 更直观地拼接变量。

## 第三方库与包管理
Linux 下推荐使用 `pip` 或 `pip3` 来安装第三方库，例如：
```bash
pip3 install requests
```
为了避免污染系统环境，通常会先创建 **虚拟环境**：
```bash
python3 -m venv venv
source venv/bin/activate
```
激活后安装的库都位于 `venv` 目录中，需要时再 `deactivate` 退出。

常见的运维相关库包括：
- **paramiko**：使用 SSH 协议远程执行命令或传输文件。
- **fabric**：基于 paramiko 的高级封装，更易批量执行任务。
- **psutil**：获取 CPU、内存、磁盘等系统信息。
- **schedule**：用 Python 直接编写定时任务脚本。
- **requests**：最常用的 HTTP 库，便于调用各类接口。
- **Flask**：轻量级 Web 框架，可快速制作简易后台页面。
- **PyYAML**：读取和生成 YAML 格式的配置文件。

示例：利用 paramiko 远程重启服务。
```python
import paramiko

ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect('192.168.1.10', username='root', password='yourpass')
stdin, stdout, stderr = ssh.exec_command('systemctl restart nginx')
print(stdout.read().decode())
ssh.close()
```

## 自动化场景示例

### 1. 批量管理服务器
假设有多台主机需要执行同一操作，可以编写脚本循环执行：
```python
hosts = ['192.168.1.10', '192.168.1.11']
for h in hosts:
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(h, username='root')
    ssh.exec_command('apt update && apt upgrade -y')
    ssh.close()
```

### 2. 监控系统状态
借助 `psutil` 获取系统资源，并根据条件触发告警：
```python
import psutil

cpu = psutil.cpu_percent(interval=1)
mem = psutil.virtual_memory().percent
if cpu > 80 or mem > 80:
    print('⚠️ 资源使用率过高')
```

### 3. 定期执行任务
除了使用 Linux 自带的 `crontab`，也可以通过 `schedule` 库以更 Pythonic 的方式安排任务：
```python
import schedule
import time

def job():
    print('执行备份……')

schedule.every().day.at('02:00').do(job)

while True:
    schedule.run_pending()
    time.sleep(1)
```

## 与运维工具结合

Python 常见的运维框架与平台包括：
- **Ansible**：通过编写 YAML 格式的 Playbook 批量管理服务器，底层使用 Python 实现。
- **SaltStack**：面向大规模集群的配置管理和自动化运维工具。
- **Fabric**：轻量级的远程执行库，适合自定义部署流程。

这些工具本身就是用 Python 编写的，因此能与 Python 脚本无缝结合，实现更加灵活的自动化流程。

## 日志与异常处理
在生产环境中，良好的日志至关重要。Python 的 `logging` 模块可以方便地记录调试信息：
```python
import logging

logging.basicConfig(
    filename='/var/log/myapp.log',
    level=logging.INFO,
    format='%(asctime)s %(levelname)s %(message)s'
)

try:
    # 关键操作
    logging.info('开始执行任务')
except Exception as e:
    logging.error('出错: %s', e)
```

同时，捕获异常并合理处理，可以避免脚本在关键时刻突然中断：
```python
try:
    risky_operation()
except Exception as err:
    print('发生错误', err)
```

## 实战：磁盘空间告警脚本
下面示例会检查磁盘使用率，并在超过阈值时发送邮件：
```python
#!/usr/bin/env python3
import shutil
import smtplib
from email.mime.text import MIMEText

threshold = 80
usage = shutil.disk_usage('/')
percent = usage.used / usage.total * 100

if percent > threshold:
    msg = MIMEText(f'Disk usage is {percent:.1f}%!')
    msg['Subject'] = 'Disk Alert'
    msg['From'] = 'monitor@example.com'
    msg['To'] = 'admin@example.com'
    with smtplib.SMTP('smtp.example.com') as s:
        s.login('user', 'pass')
        s.send_message(msg)
```
将该脚本加入 `crontab`，即可实现基础告警功能。

## 持续改进与学习建议
- **多读官方文档**：Python 官方教程和各库的文档是最佳学习资料。
- **从简单脚本开始**：先解决实际问题，再逐步抽象成可复用的函数或模块。
- **关注社区**：Stack Overflow、博客和论坛能提供丰富的案例和经验。
- **编写模块化代码**：随着脚本增多，可逐渐学习面向对象和模块化编程，便于维护。

掌握以上知识后，你就能使用 Python 高效地处理日常运维工作，从而把精力投入到更重要的任务中。祝你在运维之路越走越远！
