---
title: Shell
date: 2025-03-26 13:58:05
tags: [服务端,Shell]
categories: [服务端, Shell]
cover: https://img0.baidu.com/it/u=279259354,2980670496&fm=253&fmt=auto&app=138&f=JPEG?w=1387&h=500
---

## 什么是 Shell
Shell 是 Linux 和 Unix 系统中的 "命令行解释器"，它允许我们通过文本命令来和系统交互。最常见的 Bash 既是一门脚本语言，又提供了强大的自动化能力。

### 为什么运维要学 Shell
- **批量操作**：通过脚本一次处理多台服务器或大量文件。
- **自动化**：定时任务、监控脚本、部署脚本都离不开它。
- **成本低**：系统自带，无需额外安装。
- **可移植**：脚本在各 Linux 发行版间基本通用。

## 第一个脚本
```bash
#!/bin/bash
echo "Hello World"
```
保存为 `hello.sh`，给执行权限后运行：
```bash
chmod +x hello.sh
./hello.sh
```

## 基本语法
### 变量与字符串
```bash
name="Bob"
echo "Hello, $name"
```
- 使用 `$(command)` 可以获取命令结果：
```bash
files=$(ls /etc | wc -l)
```

### 条件判断
```bash
if [ -f /etc/passwd ]; then
  echo "文件存在"
else
  echo "文件不存在"
fi
```
支持 `-f` (文件存在)、`-d` (目录存在) 等多种条件表达式。

### 循环
#### for
```bash
for user in alice bob carol; do
  echo "欢迎 $user"
done
```
#### while
```bash
counter=0
while [ $counter -lt 5 ]; do
  echo "计数: $counter"
  counter=$((counter+1))
done
```

### 函数
```bash
backup_file() {
  src=$1
  dest=$2
  cp "$src" "$dest"
}
backup_file /etc/hosts /tmp/hosts.bak
```
函数让脚本结构更清晰，也方便复用。

## 常见运维场景
### 批量执行命令
借助 `for` 循环和 `ssh` 可以对多台服务器批量操作：
```bash
servers="192.168.1.10 192.168.1.11"
for host in $servers; do
  ssh root@$host "systemctl restart nginx"
done
```

### 日志监控
使用 `tail -f` 实时查看日志并配合条件触发告警：
```bash
tail -f /var/log/nginx/access.log | while read line; do
  echo "$line" | grep "500" && echo "发现 500 错误"
done
```

### 自动备份
```bash
#!/bin/bash
src=/data
bak=/backup/$(date +%F)
mkdir -p "$bak"
cp -r "$src" "$bak"
```
搭配 `crontab` 定时执行即可完成每日备份。

## 自动化技巧
1. **结合 crontab 定时运行脚本**：
   ```bash
   # 每天凌晨 2 点备份
   0 2 * * * /usr/local/bin/backup.sh
   ```
2. **环境变量管理**：在脚本开头使用 `export` 设置变量，或加载 `.env` 文件。
3. **日志记录**：`exec >>/var/log/myscript.log 2>&1` 可把脚本输出写入日志文件。
4. **使用 set -euo pipefail 提高健壮性**：
   ```bash
   #!/bin/bash
   set -euo pipefail
   ```

## 实战示例：一键部署 Web 服务
```bash
#!/bin/bash
set -e
app_src=/opt/app
web_root=/var/www/html

# 安装依赖
apt update && apt install -y nginx git

# 拉取代码并部署
if [ ! -d "$app_src" ]; then
  git clone https://example.com/repo.git "$app_src"
else
  git -C "$app_src" pull
fi
cp -r "$app_src"/* "$web_root"/

# 启动服务
systemctl restart nginx
```

## 学习建议
- 每天写一点脚本，练习常用命令。
- 阅读他人的脚本，理解其中的逻辑。
- 遇到问题先试着手动操作，再用脚本封装。

Shell 是一把瑞士军刀，为 Linux 运维带来无限可能。熟练掌握后，你可以把重复工作交给脚本，专注更高价值的任务。

## 扩展概念
- **Shell 与终端**：终端 (Terminal) 是显示和输入界面，Shell 是真正解析命令的程序。常见 Shell 有 Bash、Zsh 等。
- **Shebang**：脚本第一行 `#!/bin/bash` 指定用哪个解释器运行，这是让脚本能够直接执行的关键。
- **交互式与非交互式**：登录后手动输入命令属于交互式，定时任务或脚本调用则是非交互式，需要确保变量与环境正确。
- **管道与重定向**：使用 `|` 把多个命令连接成流水线，`>` 或 `>>` 把输出写入文件，是构建复杂流程的核心方式。

## 企业常用脚本案例
### 1. 服务健康检查
```bash
#!/bin/bash
url="https://example.com/health"
status=$(curl -s -o /dev/null -w "%{http_code}" "$url")
if [ "$status" != "200" ]; then
  echo "[警告] 服务异常，状态码 $status" | mail -s "Health Check" admin@example.com
fi
```

### 2. 自动清理日志
```bash
#!/bin/bash
logdir=/var/log/myapp
find "$logdir" -name "*.log" -mtime +7 -delete
```

### 3. 批量发布代码
```bash
servers="srv1 srv2 srv3"
for h in $servers; do
  ssh $h "cd /opt/app && git pull && systemctl restart app"
done
```

### 4. 数据备份同步
```bash
#!/bin/bash
src=/data
remote=user@backup:/data
rsync -avz "$src" "$remote"
```

企业脚本往往围绕监控、备份、部署、清理等场景展开。实践中可以根据需要组合成完整的自动化流程。
