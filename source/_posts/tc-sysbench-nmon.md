---
title: 数据库高可用测试工具使用指南：tc / sysbench / nmon
categories:
  - 数据库
tags:
  - tc
  - sysbench
  - nmon
  - 数据库
  - 高可用
description: 信创集中式数据库高可用测试中三种工具的使用：tc 模拟同城网络延迟、sysbench 数据库压测、nmon 系统资源监控，含安装、参数、命令示例与结果解读。
top: false
abbrlink: dc3a542e
date: 2026-09-04 09:30:00
---

在信创集中式数据库高可用测试中，三种工具各司其职：

| 工具 | 用途 | 在本次测试中的角色 |
| :--- | :--- | :--- |
| **tc** | Linux 流量控制 | 模拟同城双机房之间的 1.5ms 网络延迟 |
| **sysbench** | 数据库压测 | 预置数据 + 只读/只写/读写混合场景压测、长稳 |
| **nmon** | 系统资源监控 | 采集 CPU / 内存 / 磁盘 I/O |

---

## 一、tc —— 模拟同城网络延迟

### 1.1 原理

`tc`（Traffic Control）是 Linux 内核的流量控制工具，通过 **qdisc（队列规则）** 对网卡出方向流量做整形。其中 `netem`（Network Emulator）qdisc 专门用来模拟延迟、丢包、乱序、抖动等网络异常。

> 注意：`netem` 只对**出方向**生效。要模拟主备之间的双向延迟，通常需要在两个节点上分别对"发往对方"的流量加延迟。

### 1.2 基本用法

```bash
# 查看网卡现有规则
tc qdisc show dev eth0

# 对 eth0 所有出方向流量加 1.5ms 延迟
tc qdisc add dev eth0 root netem delay 1.5ms

# 修改延迟（不删重建，直接改）
tc qdisc change dev eth0 root netem delay 5ms

# 删除规则、恢复原状
tc qdisc del dev eth0 root

# 模拟丢包 1%
tc qdisc add dev eth0 root netem loss 1%

# 延迟 + 抖动（50ms ± 10ms）
tc qdisc add dev eth0 root netem delay 50ms 10ms
```

### 1.3 只对指定主机加延迟（filter 分类）

数据库节点上通常不能给所有流量都加延迟，只对"发往对端节点"的流量加。用 `htb` 分类 + `u32` 过滤器实现：

```bash
# 清空
tc qdisc del dev eth0 root 2>/dev/null

# 根 qdisc 用 htb，未匹配流量走默认类 1:2（无延迟）
tc qdisc add dev eth0 root handle 1: htb default 2
tc class add dev eth0 parent 1: classid 1:2 htb rate 1000mbit

# 延迟类 1:1（给目标 IP 加 1.5ms）
tc class add dev eth0 parent 1: classid 1:1 htb rate 1000mbit
tc qdisc add dev eth0 parent 1:1 handle 10: netem delay 1.5ms

# 匹配发往 192.168.1.20（对端节点）的流量，走延迟类
tc filter add dev eth0 protocol ip parent 1: prio 1 u32 \
    match ip dst 192.168.1.20/32 flowid 1:1
```

在另一台节点上对称配置（目标 IP 换成对方）即可模拟双向延迟。

### 1.4 常用命令速查

| 需求 | 命令 |
| :--- | :--- |
| 全部流量加延迟 | `tc qdisc add dev eth0 root netem delay 1.5ms` |
| 指定 IP 加延迟 | 见 1.3 filter 方式 |
| 临时调整 | `tc qdisc change dev eth0 root netem delay 5ms` |
| 撤销 | `tc qdisc del dev eth0 root` |

> 前置：需 root 权限；内核需支持 `netem` 模块（`modprobe sch_netem`，一般发行版默认已编入）。

---

## 二、sysbench —— 数据库压测

### 2.1 安装

```bash
# Ubuntu / Debian
apt install sysbench

# CentOS / RHEL
yum install sysbench
```

数据库压测用的是 `oltp.lua`（另有 `select.lua` 非事务读测试）。脚本通常在 `/usr/share/sysbench/` 下，也可拷贝到本地 `./lua/` 目录自行修改（各数据库按需修改 `common.lua` 适配自身语法）。

### 2.2 三个阶段

```bash
sysbench <参数> prepare    # 建表 + 造数据
sysbench <参数> run        # 执行压测
sysbench <参数> cleanup    # 清空数据（测完清理）
```

### 2.3 常用参数（MySQL 协议）

| 参数 | 含义 | 本次取值 |
| :--- | :--- | :--- |
| `--mysql-host` / `--mysql-port` | 连接地址 / 端口 | 127.0.0.1 / 6001 |
| `--mysql-user` / `--mysql-password` | 账号 / 密码 | superuser / xxxx |
| `--mysql-db` | 库名 | ndm |
| `--oltp_tables_count` | 表数量 | 10 |
| `--oltp_table_size` | 每表行数 | 10000000（1000 万） |
| `--num-threads` | 并发线程数 | 16（长稳用 60） |
| `--max-time` | 运行时长（秒） | 60（长稳 86400） |
| `--max-requests` | 请求总数上限（0 为不限） | 0 |
| `--report-interval` | 间隔报告秒数 | 3 |
| `--percentile` | 输出的延迟分位数 | 95 / 60 |
| `--oltp-read-only` | 是否只读 | on / off |
| `--test` | 指定 lua 脚本 | ./lua/oltp.lua |

`--oltp_*` 操作比例参数（决定混合场景读写配比）：

| 参数 | 说明 |
| :--- | :--- |
| `--oltp_point_selects` | 点查次数 |
| `--oltp_simple_ranges` | 简单范围查询次数 |
| `--oltp_sum_ranges` / `--oltp_order_ranges` / `--oltp_distinct_ranges` | 聚合/排序/去重范围查询 |
| `--oltp_index_updates` / `--oltp_non_index_updates` | 索引列 / 非索引列更新 |
| `--oltp_delete_inserts` | 删除+插入 |

### 2.4 三种场景命令

```bash
# 只读：point_selects=8 + simple_ranges=2，无任何写
./sysbench --mysql-host=127.0.0.1 --mysql-port=6001 --mysql-user=superuser \
  --mysql-password=xxxx --mysql-db=ndm \
  --oltp_tables_count=10 --oltp_table_size=10000000 \
  --num-threads=16 --report-interval=3 --max-time=60 --max-requests=0 \
  --percentile=95 --test=./lua/oltp.lua --oltp-read-only=on \
  --oltp_range_size=100 --oltp_point_selects=8 --oltp_simple_ranges=2 \
  --oltp_sum_ranges=0 --oltp_order_ranges=0 --oltp_distinct_ranges=0 \
  --oltp_index_updates=0 --oltp_non_index_updates=0 --oltp_delete_inserts=0 run

# 只写：读操作全为 0，index_updates=2 + non_index_updates=1 + delete_inserts=1
./sysbench --mysql-host=127.0.0.1 --mysql-port=6001 --mysql-user=superuser \
  --mysql-password=xxxx --mysql-db=ndm \
  --oltp_tables_count=10 --oltp_table_size=10000000 \
  --num-threads=16 --report-interval=3 --max-time=60 --max-requests=0 \
  --percentile=95 --test=./lua/oltp.lua --oltp-read-only=off \
  --oltp_range_size=100 --oltp_point_selects=0 --oltp_simple_ranges=0 \
  --oltp_sum_ranges=0 --oltp_order_ranges=0 --oltp_distinct_ranges=0 \
  --oltp_index_updates=2 --oltp_non_index_updates=1 --oltp_delete_inserts=1 run

# 读写混合：point_selects=8 + simple_ranges=2 + index_updates=2
./sysbench --mysql-host=127.0.0.1 --mysql-port=6001 --mysql-user=superuser \
  --mysql-password=xxxx --mysql-db=ndm \
  --oltp_tables_count=10 --oltp_table_size=10000000 \
  --num-threads=16 --report-interval=3 --max-time=60 --max-requests=0 \
  --percentile=60 --test=./lua/oltp.lua --oltp-read-only=off \
  --oltp_range_size=100 --oltp_point_selects=8 --oltp_simple_ranges=2 \
  --oltp_sum_ranges=0 --oltp_order_ranges=0 --oltp_distinct_ranges=0 \
  --oltp_index_updates=2 run
```

### 2.5 结果解读

关键输出段：

```text
SQL statistics:
    queries performed:
        read:     ...     write:    ...     other:    ...     total:    ...
    transactions:         ...   (per sec.)
    queries:              ...   (per sec.)
    ...
Latency (ms):
         min:  ...   avg:  ...   max:  ...   95th percentile:  ...
```

| 指标 | 含义 |
| :--- | :--- |
| **TPS** | `transactions per sec.`，每秒事务数，写场景核心指标 |
| **QPS** | `queries per sec.`，每秒 SQL 数，读场景核心指标 |
| **95th percentile** | 95% 请求的延迟低于该值，看尾部延迟 |

> 长稳测试把 `--max-time` 设为 86400（24 小时）、`--num-threads` 设为 60，`--oltp-read-only` 按事务/非事务脚本区分。

### 2.6 适配信创库：修改 common.lua（以 Kingbase/PG 为例）

`oltp.lua` 默认按 MySQL 语法生成 DDL/DML。若目标库为 PG 协议（如人大金仓 Kingbase），需改 `common.lua` 里的建表与写入语句；若目标库提供 MySQL 兼容协议（可直接用 `--mysql-*` 连），则无需修改。

主要改动点（MySQL → PG）：

| 位置 | MySQL 写法 | PG / Kingbase 写法 |
| :--- | :--- | :--- |
| 主键自增 | `id INTEGER NOT NULL AUTO_INCREMENT` | `id SERIAL PRIMARY KEY`（或 `GENERATED ALWAYS AS IDENTITY`） |
| 表选项 | `ENGINE=InnoDB` | 删除（PG 无引擎概念） |
| 索引 | 内联 `KEY k_1 (c1)` | 独立 `CREATE INDEX k_1 ON sbtest1(c1)` |
| 覆盖写 | `REPLACE INTO ...` | `INSERT ... ON CONFLICT (id) DO UPDATE SET ...` |

典型改动示例（`common.lua`）：

```lua
-- 建表（create_table 部分）
-- 原： "id INTEGER NOT NULL AUTO_INCREMENT," ..
--      "PRIMARY KEY (id)," ..
-- 改： "id SERIAL PRIMARY KEY," ..
-- 去掉 "ENGINE = InnoDB"；内联 KEY 改为独立 CREATE INDEX

-- 覆盖写（DELETE+INSERT 场景）
-- 原："REPLACE INTO sbtest%u (id, k, c, pad) VALUES (...)"
-- 改："INSERT INTO sbtest%u (id, k, c, pad) VALUES (...)
--      ON CONFLICT (id) DO UPDATE SET c=EXCLUDED.c, pad=EXCLUDED.pad"
```

> 不同信创库语法细节有差异，以各自官方文档为准；改完先在单表 `prepare` 验证 DDL 能成功执行，再 `run`。

---

## 三、nmon —— 系统资源监控

### 3.1 安装

```bash
# Ubuntu / Debian
apt install nmon

# CentOS / RHEL
yum install nmon
```

### 3.2 实时交互模式

直接运行 `nmon`，按键盘切换视图：

| 按键 | 视图 |
| :--- | :--- |
| `c` | CPU |
| `m` | 内存 |
| `d` | 磁盘 I/O |
| `n` | 网络 |
| `q` | 退出 |

适合快速看当前状态；压测要留档需用下面的采集模式。

### 3.3 采集模式（后台落盘）

```bash
mkdir -p /data/nmon
nmon -f -s 60 -c 1440 -m /data/nmon
```

| 参数 | 含义 | 本次取值 |
| :--- | :--- | :--- |
| `-f` | 后台模式，输出到文件 | — |
| `-s 60` | 采样间隔（秒） | 60 |
| `-c 1440` | 采样次数 | 1440（= 24 小时） |
| `-m` | 输出目录 | /data/nmon |

生成文件名为 `主机名_日期_时间.nmon`（如 `node1_20260904_0900.nmon`）。

### 3.4 结果分析

- **nmon_analyser**（官方 Excel 宏）：导入 `.nmon` 文件，自动生成 CPU/内存/磁盘图表，最常用；
- **nmonchart**（命令行）：把 `.nmon` 转成 HTML 图表；
- **grep 手工取数**：
  ```bash
  grep '^CPU_ALL' /data/nmon/*.nmon   # CPU 数据
  grep '^MEM'     /data/nmon/*.nmon   # 内存数据
  grep '^DISK'    /data/nmon/*.nmon   # 磁盘 I/O 数据
  ```

### 3.5 关键指标与判定

| 指标 | 来源行 | 判定参考 |
| :--- | :--- | :--- |
| CPU 使用率 | `CPU_ALL` 的 `User% + Sys%` | 平均 ≤ 60% |
| 内存泄露 | `MEM` 的 `MemFree` 持续下降 / `MemUsed` 持续上升 | 单调上升不回落 = 泄露 |
| 磁盘 I/O | `DISKREAD` / `DISKWRITE`（或 `DISKBUSY`） | 记录存档，无硬性阈值 |

---

## 四、三者在本测试中的配合

```text
┌──────────┐   tc 1.5ms   ┌──────────┐
│ 主节点 A  │ ◄──────────► │ 备节点 B  │     tc：模拟同城延迟
└────┬─────┘              └──────────┘
     │  sysbench 压测（只读/只写/混合、长稳）
     │  nmon 采集 CPU/内存/磁盘 I/O
     ▼
```

1. **准备阶段**：`sysbench ... prepare` 预置 10 表 × 1000 万行；
2. **环境模拟**：两个节点上各用 `tc` 对"发往对方"的流量加 1.5ms 延迟；
3. **压测阶段**：`sysbench ... run` 跑三种场景，同时 `nmon -f -s 60 -c 1440` 采集资源；
4. **分析阶段**：nmon 图表看 CPU ≤ 60%、内存是否泄露；sysbench 输出记录 TPS/QPS/延迟基线；
5. **清理**：`sysbench ... cleanup` 清数据，`tc qdisc del dev eth0 root` 撤销延迟。
