---
title: Shell 脚本快速入门（小白总结）
categories:
  - 编程
tags:
  - Shell
  - Linux
  - 自动化脚本
  - 学习笔记
description: 面向自动化测试/运维小白的 Shell 脚本快速入门，覆盖变量、条件判断、循环、函数、重定向、文本三剑客等核心语法，附数据库测试实战脚本，快速上手写脚本。
top: false
abbrlink: 39af0d1b
date: 2026-10-20 10:30:00
---

> **文章目的**：面向需要在 Linux 上写脚本做自动化测试的小白，意在快速入门、能看懂并独立编写测试脚本。本文覆盖 Shell 核心语法 + 常用命令，看完即可上手写批量任务、备份、日志分析等脚本。

## 学习平台

| 平台 | 说明 |
|------|------|
| [菜鸟教程 Shell](https://www.runoob.com/linux/linux-shell.html) | 语法速查，忘了翻一眼 |
| [Linux 就该这么学](https://www.linuxprobe.com/) | 中文系统教程，适合入门 |
| `man <命令>` / `<命令> --help` | 系统自带手册，最权威 |

---

## 一、Shell 是什么 & 第一个脚本

Shell 是 Linux 的**命令行解释器**，你敲的每条命令都是在和 Shell 交互。把命令写进文件，就变成了**脚本**，可以批量、重复执行——这正是自动化测试要的。

一个最简单的脚本：

```bash
#!/bin/bash
# 这是我的第一个脚本

echo "Hello, Shell!"
name="张三"
echo "你好，${name}"
```

| 要点 | 说明 |
|------|------|
| `#!/bin/bash` | shebang，声明用哪个解释器执行，必须放第一行 |
| `#` | 注释，Shell 里没有多行注释 |
| `echo` | 打印输出 |

两种运行方式：

```bash
# 方式一：直接执行（需要可执行权限）
chmod +x test.sh     # 加执行权限
./test.sh

# 方式二：用 bash 执行（不需要可执行权限，最常用）
bash test.sh
```

> 💡 写脚本建议统一用 `bash 脚本名.sh` 运行，省去加权限的麻烦；脚本文件后缀 `.sh` 只是约定，不是必须。

---

## 二、变量

### 1. 定义与引用

```bash
name="zhangsan"     # 定义变量：等号两边【不能】有空格
age=25              # 数字也是字符串

echo $name          # 引用变量 -> zhangsan
echo ${name}        # 花括号形式更安全，明确边界
echo "${name}是${age}岁"   # 字符串拼接

readonly PI=3.14    # 只读变量，不能修改
unset name          # 删除变量
```

> ⚠️ **新手最易错**：`name = "zhangsan"`（等号两边有空格）会报错。赋值时等号两边**紧贴**。

### 2. 特殊变量（脚本必会）

| 变量 | 含义 |
|------|------|
| `$0` | 脚本名本身 |
| `$1` ~ `$9` | 第 1~9 个参数 |
| `$#` | 参数个数 |
| `$@` | 所有参数（各自独立） |
| `$*` | 所有参数（拼成一个整体） |
| `$?` | 上一条命令的退出码，**0 表示成功** |
| `$$` | 当前进程 PID |
| `$!` | 最后一个后台进程 PID |

```bash
#!/bin/bash
echo "脚本名：$0"
echo "第一个参数：$1"
echo "参数个数：$#"
echo "所有参数：$@"
```

> `$?` 是判断命令成败的关键，后面条件判断、测试脚本断言全靠它。

### 3. 命令替换与环境变量

```bash
now=$(date)         # 把命令结果存进变量（推荐写法）
# now=`date`        # 反引号旧写法，效果相同

echo "当前时间：$now"

# 环境变量（系统已定义，直接读）
echo $PATH          # 命令搜索路径
echo $HOME          # 家目录
echo $USER          # 当前用户

# 自定义变量默认只在当前脚本内有效，导出后子进程才能用
export MY_VAR="hello"
```

---

## 三、字符串操作

```bash
str="hello world"

echo ${#str}            # 长度 -> 11
echo ${str:0:5}         # 截取，从 0 开始取 5 个 -> hello
echo ${str/world/中国}   # 替换 -> hello 中国
```

单引号 vs 双引号（重要）：

```bash
name="张三"
echo '你好 $name'   # 单引号：原样输出 -> 你好 $name
echo "你好 $name"   # 双引号：解析变量 -> 你好 张三
```

> 💡 记一句：**单引号不解析，双引号解析变量**。要输出 `$` 本身或纯文本用单引号。

---

## 四、数组

```bash
arr=(apple banana cherry)   # 定义，空格分隔

echo ${arr[0]}      # 第一个 -> apple
echo ${arr[1]}      # 第二个 -> banana
echo ${arr[@]}      # 所有元素
echo ${#arr[@]}     # 元素个数 -> 3

arr[3]=date         # 追加/修改
```

> 🎯 **实战**：数据库集群的节点 IP 列表用数组存，遍历即可批量操作（见文末实战）。

---

## 五、条件判断

### 1. if 语句

```bash
score=85

if [ $score -ge 90 ]; then
    echo "优秀"
elif [ $score -ge 60 ]; then
    echo "及格"
else
    echo "不及格"
fi                          # if 结束必须写 fi
```

> ⚠️ `[ ]` 方括号**内侧两边都要有空格**：`[ $score -ge 90 ]` 正确，`[$score -ge 90]` 报错。

### 2. 判断条件速查

数字比较：

| 写法 | 含义 |
|------|------|
| `-eq` | 等于 |
| `-ne` | 不等于 |
| `-gt` / `-ge` | 大于 / 大于等于 |
| `-lt` / `-le` | 小于 / 小于等于 |

字符串比较：

| 写法 | 含义 |
|------|------|
| `=` / `==` | 相等 |
| `!=` | 不等 |
| `-z` | 长度为 0（空） |
| `-n` | 长度非 0（非空） |

文件判断（测试脚本常用）：

| 写法 | 含义 |
|------|------|
| `-e` | 文件存在 |
| `-f` | 是普通文件 |
| `-d` | 是目录 |
| `-r` / `-w` / `-x` | 可读 / 可写 / 可执行 |
| `-s` | 文件非空 |

```bash
# 参数为空判断
if [ -z "$1" ]; then
    echo "用法：$0 <文件名>"
    exit 1
fi

# 文件存在判断
if [ -f /etc/passwd ]; then
    echo "文件存在"
fi
```

### 3. 进阶：`[[ ]]` 与逻辑运算

```bash
# [[ ]] 是 bash 增强版，支持 && || 和正则
if [[ $1 == "start" || $1 == "stop" ]]; then
    echo "合法命令"
fi

# 正则匹配（=~）
if [[ "$ip" =~ ^192\.168\. ]]; then
    echo "是内网 IP"
fi
```

---

## 六、循环

### 1. for 循环

```bash
# 遍历列表
for i in 1 2 3 4 5; do
    echo $i
done

# 遍历序列
for i in {1..10}; do
    echo $i
done

# 遍历文件（最常用：批量处理）
for file in *.sh; do
    echo "处理 $file"
done

# C 语言风格
for ((i=0; i<5; i++)); do
    echo $i
done
```

### 2. while / until 循环

```bash
# while：条件成立时循环
count=1
while [ $count -le 3 ]; do
    echo "第 $count 次"
    count=$((count + 1))     # 注意：算术运算要用 $(( ))
done

# until：条件不成立时循环（直到成立才停）
count=1
until [ $count -gt 3 ]; do
    echo "第 $count 次"
    count=$((count + 1))
done
```

> 💡 Shell 里做算术用 `$(( ))`，不能用 `count + 1`。

### 3. case 语句（多分支，比一堆 if 清爽）

```bash
case $1 in
    start)
        echo "启动..."
        ;;
    stop)
        echo "停止..."
        ;;
    restart)
        echo "重启..."
        ;;
    *)
        echo "用法：$0 {start|stop|restart}"
        exit 1
        ;;
esac
```

### 4. 循环控制

```bash
break       # 跳出整个循环
continue    # 跳过本次，继续下一次
```

---

## 七、函数

```bash
# 定义函数
say_hello() {
    local name=$1      # local 声明局部变量
    echo "你好，$name"
    return 0           # return 返回状态码（0 成功）
}

# 调用：直接写函数名，参数空格隔开
say_hello 张三          # -> 你好，张三

# 函数内 $1 是参数，$? 是返回值
add() {
    echo $(( $1 + $2 ))
}
result=$(add 3 5)      # 用命令替换接住结果 -> 8
```

> 💡 Shell 函数没有 Python 那样的 `return 值`，`return` 只返回状态码（0/非0）；要返回数据用 `echo` + 命令替换接住。

---

## 八、重定向与管道

| 写法 | 含义 |
|------|------|
| `cmd > file` | 标准输出覆盖写到文件 |
| `cmd >> file` | 标准输出追加到文件 |
| `cmd 2> file` | 错误输出写到文件 |
| `cmd &> file` | 标准输出和错误一起写 |
| `cmd < file` | 从文件读输入 |
| `cmd1 \| cmd2` | 管道：把 cmd1 输出作为 cmd2 输入 |
| `/dev/null` | 「黑洞」，丢弃输出 |

```bash
echo "hello" > out.txt        # 覆盖写
echo "world" >> out.txt       # 追加写
ls /nonexist 2> err.txt       # 错误重定向
ls /nonexist 2> /dev/null     # 丢弃错误信息

# 管道链：查看进程 -> 过滤 -> 计数
ps -ef | grep nginx | wc -l
```

> 🎯 测试脚本里常用 `命令 > /dev/null 2>&1` 静默执行，只用 `$?` 判断成败。

---

## 九、常用命令速查

### 1. 文件与目录

| 命令 | 作用 | 常用示例 |
|------|------|----------|
| `ls` | 列目录 | `ls -lh`（带大小）、`ls -lrt`（按时间倒序） |
| `cd` / `pwd` | 切换 / 显示当前目录 | `cd /data && pwd` |
| `cp` / `mv` / `rm` | 复制 / 移动 / 删除 | `cp -r a/ b/`、`rm -rf dir/` |
| `mkdir` | 建目录 | `mkdir -p a/b/c`（递归创建） |
| `touch` | 建空文件 | `touch a.txt` |
| `cat` / `head` / `tail` | 查看文件 | `head -n 10`、`tail -n 20` |
| `tail -f` | 实时跟踪日志 | `tail -f app.log`（测试看日志必备） |
| `find` | 查找文件 | `find / -name "*.log"` |
| `wc` | 统计 | `wc -l`（行数） |

### 2. 文本三剑客：grep / sed / awk

**grep** —— 过滤查找：

```bash
grep "ERROR" app.log          # 找含 ERROR 的行
grep -i "error" app.log       # 忽略大小写
grep -v "DEBUG" app.log       # 反选（排除）
grep -c "ERROR" app.log       # 计数
grep -r "keyword" /data/logs/ # 递归搜目录
grep -E "error|warn" app.log  # 正则（多关键词）
```

**sed** —— 查找替换、按行操作：

```bash
sed 's/old/new/' file         # 每行替换第一个
sed 's/old/new/g' file        # 全局替换
sed -i 's/old/new/g' file     # 原地修改（写回文件）
sed -n '5,10p' file           # 打印第 5~10 行
sed '/error/d' file           # 删除含 error 的行
```

**awk** —— 按列处理（日志分析神器）：

```bash
awk '{print $1, $3}' file              # 打印第 1、3 列（默认空格分隔）
awk -F: '{print $1}' /etc/passwd       # -F 指定分隔符
awk '$3 > 100 {print $1}' file         # 第 3 列大于 100 才打印
awk '{sum += $1} END {print sum}' file # 第 1 列求和
```

### 3. 进程与资源

| 命令 | 作用 |
|------|------|
| `ps -ef` | 查看所有进程 |
| `top` | 实时资源监控（`q` 退出） |
| `kill` / `kill -9` | 结束进程 |
| `nohup cmd &` | 后台运行，退出终端不中断 |
| `df -h` / `du -sh` | 磁盘使用 |
| `free -h` | 内存使用 |

### 4. 权限、网络、压缩

```bash
chmod +x script.sh       # 加执行权限
chmod 755 script.sh      # 数字法：rwxr-xr-x
chown user:group file    # 改属主

ping -c 3 192.168.1.10   # ping 3 次
ss -tlnp                 # 查看监听端口
curl -I https://example.com   # 查看响应头

tar -czf bak.tar.gz dir/ # 打包压缩
tar -xzf bak.tar.gz      # 解压
```

---

## 十、新手易错点

1. **等号两边不能有空格**：`a=1` 对，`a = 1` 错。
2. **`[ ]` 内侧要有空格**：`[ $a -eq 1 ]` 对，`[$a -eq 1]` 错。
3. **单引号 vs 双引号**：单引号原样输出，双引号解析变量。
4. **算术要用 `$(( ))`**：没有 `i++`，用 `i=$((i+1))`。
5. **`$?` 为 0 才代表成功**：非 0 是失败，判断命令是否执行成功看它。
6. **Windows 换行符坑**：在 Windows 上用记事本编辑脚本，换行是 `\r\n`，Linux 会报 `\r` 错误。解决：`dos2unix script.sh`，或在 Vim 里 `:set ff=unix` 保存。
7. **`set -e` 让脚本出错即停**：加上后任一命令失败立即退出，测试脚本推荐加。

```bash
#!/bin/bash
set -e          # 出错立即退出
set -u          # 使用未定义变量报错
set -x          # 调试模式，打印每条执行的命令（排查用，用完记得删）
```

---

## 十一、实战脚本（结合数据库测试）

### 1. 批量检查集群节点连通性

```bash
#!/bin/bash
# 检查金仓数据库集群各节点是否 ping 通
nodes=(192.168.1.10 192.168.1.11 192.168.1.12)

for ip in "${nodes[@]}"; do
    if ping -c 1 -W 1 "$ip" > /dev/null 2>&1; then
        echo "[OK]   $ip 可达"
    else
        echo "[FAIL] $ip 不通"
    fi
done
```

### 2. 数据库定时备份脚本

```bash
#!/bin/bash
# 每天备份数据库，文件名带日期，保留最近 7 天
DB_NAME="ndm"
BACKUP_DIR="/data/backup"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# 金仓/PostgreSQL 协议用 sys_dump / pg_dump，按实际改
pg_dump "$DB_NAME" > "$BACKUP_DIR/${DB_NAME}_${DATE}.sql"

# 删除 7 天前的旧备份
find "$BACKUP_DIR" -name "*.sql" -mtime +7 -delete

echo "备份完成：${DB_NAME}_${DATE}.sql"
```

> 配上 crontab 定时任务：`0 2 * * * /data/backup.sh`，每天凌晨 2 点自动备份。

### 3. 日志错误分析

```bash
# 统计日志中各类错误出现次数
grep -oE "ERROR|WARN|FATAL" app.log | sort | uniq -c

# 找出响应时间最慢的 10 条请求（假设第 3 列是耗时毫秒）
awk '{print $3, $0}' app.log | sort -nr | head -10
```

---

## 下一步

掌握以上语法和命令，就可以：

- **写批量任务**：批量建目录、批量改名、批量执行命令
- **做日志分析**：grep/awk 提取测试结果、统计错误
- **写部署/备份脚本**：配合 crontab 定时执行
- **看 Linux 教程**：进阶可学 `expect`（交互式自动化）、`jq`（处理 JSON）

> 本文持续更新中，欢迎在评论区交流。
