---
title: Python 基础语法快速入门（小白总结）
categories:
  - 编程
tags:
  - Python
  - 自动化测试
  - 学习笔记
description: 面向自动化测试小白的 Python 基础语法速查总结，覆盖变量、数据类型、控制流、函数、类、异常处理等核心语法，快速上手搭建测试框架。
top: false
abbrlink: 435f797f
date: 2026-08-19 20:20:11
---

> **文章目的**：面向需要用 Python 搭建测试框架的小白，意在快速入门、实操自动化框架搭建。本文是第一篇——基础语法总结，看完即可读懂和编写测试脚本。

## 学习平台

| 平台 | 说明 |
|------|------|
| [Python123](https://www.python123.io) | 中文交互式教程，适合入门边学边练 |
| [Python 官方中文教程](https://docs.python.org/zh-cn/3/tutorial/) | 权威文档，适合系统查阅 |
| [菜鸟教程](https://www.runoob.com/python3/python3-tutorial.html) | 语法速查手册，忘了翻一眼 |

---

## 一、变量与数据类型

Python 是**动态类型**语言，变量无需声明类型，赋值即创建。

```python
name = "张三"        # 字符串 str
age = 25             # 整数 int
height = 1.75        # 浮点数 float
is_student = True    # 布尔 bool
score = None         # 空值 NoneType（类似其他语言的 null）
```

常用类型转换：

```python
int("25")     # 25  字符串转整数
str(25)       # "25"  整数转字符串
float("1.5")  # 1.5
bool(0)       # False，0、空串、空列表等都为 False
```

> 💡 检查类型用 `type(x)`，判断类型用 `isinstance(x, int)`。

---

## 二、字符串

```python
s = "hello world"

# 格式化（推荐 f-string，Python 3.6+）
name = "张三"
print(f"你好，{name}！")          # 你好，张三！

# 常用方法
s.upper()          # 转大写
s.lower()          # 转小写
s.split(" ")       # 按空格分割成列表
s.strip()          # 去掉首尾空白
s.replace("h", "H")  # 替换
s.startswith("he")   # 是否以某前缀开头
"a" in s           # 判断是否包含，返回 True/False
```

---

## 三、容器类型（重点）

测试框架里处处要用，务必熟练掌握。

### 1. 列表 list（有序、可变）

```python
nums = [1, 2, 3]
nums.append(4)       # 末尾添加 -> [1,2,3,4]
nums.insert(0, 0)    # 指定位置插入 -> [0,1,2,3,4]
nums.pop()           # 弹出末尾元素
nums[0]              # 下标访问，从 0 开始
nums[-1]             # 倒数第一个
nums[1:3]            # 切片，取下标 1~2

# 遍历
for n in nums:
    print(n)

# 列表推导式（Python 精髓，务必会）
squares = [x**2 for x in range(5)]   
# [0, 1, 4, 9, 16]   # 0, 1, 2, 3, 4的平方

# 还可以添加if进行条件筛选，结构扩展为  [ 表达式 for 变量 in 序列 if 条件 ]
```


🎯 编程训练

**题目**：给定分数列表 `scores = [85, 92, 78, 95, 55, 88, 60, 99, 42]`，用列表推导式完成：

1. 找出所有及格（≥ 60）的分数
2. 给每个分数加 5 分（模拟加分）

<details>
<summary>参考答案（点击展开）</summary>

```python
scores = [85, 92, 78, 95, 55, 88, 60, 99, 42]

# 1. 找出及格分数
passed = [s for s in scores if s >= 60]
print(passed)   # [85, 92, 78, 95, 88, 60, 99]

# 2. 每个分数加 5 分
bonus = [s + 5 for s in scores]
print(bonus)    # [90, 97, 83, 100, 60, 93, 65, 104, 47]
```

</details>

### 2. 元组 tuple（有序、不可变）

元组用小括号 `()` 定义，和列表很像，但**创建后不能增删改**，适合存放固定不变的数据（如坐标、配置项）。

```python
point = (3, 4)          # 定义元组
single = (5,)           # 单个元素必须加逗号，否则 (5) 是普通整数
x, y = point            # 解包赋值 -> x=3, y=4

point[0]                # 下标访问 -> 3
len(point)              # 长度 -> 2
point.count(3)          # 统计元素 3 出现次数
point.index(4)          # 元素 4 的下标 -> 1

# 元组不可变，下面的操作会报错
# point[0] = 10         # TypeError: 'tuple' object does not support item assignment
```

> 🎯 **实战**：自动化测试里，金仓数据库主备集群 2 个节点的节点列表常用元组存放——节点 IP、端口固定不变，用 `namedtuple` 给字段起名，比 `node[0]` 直观：

```python
from collections import namedtuple

Node = namedtuple("Node", ["ip", "port", "role"])   # ip、端口、角色（主/备）

cluster = (
    Node("192.168.1.10", 54321, "primary"),
    Node("192.168.1.11", 54321, "standby"),
)

for node in cluster:                # 遍历连接各节点
    print(f"连接 {node.role} 节点：{node.ip}:{node.port}")

primary = cluster[0]                # 主节点固定放第一个
standby  = cluster[1]               # 备节点
```

> 用元组还是列表：关键看角色会不会变，不变的用元组，变长用列表。
> 注意：若需**主备切换**（failover 后主备角色互换），角色会变，此时改用列表 + 字典更合适。

### 3. 字典 dict（键值对，最常用）

字典用花括号 `{}` 定义，由「键: 值」对组成，通过键快速访问值——接口测试里的 JSON 数据、用例参数化都靠它。

```python
user = {"name": "张三", "age": 25}
user["name"]         # 访问 -> "张三"
user["city"] = "北京"  # 添加/修改
user.get("gender", "未知")  # 安全访问，键不存在时返回默认值

# 遍历
for key, value in user.items():
    print(key, value)
# 方法	          返回	    适用场景
# user.keys()	只遍历键	只要键时
# user.values()	只遍历值	只要值时
# user.items()	键和值一起（元组）	两个都要时（最常用）

# 判断键是否存在
if "name" in user:
    print(user["name"])
```

> 🎯 **实战**：接上一节的金仓主备集群，若需**主备切换**（failover 后角色互换），角色可变，改用「列表 + 字典」存放，切换时直接改 `role` 即可：

```python
cluster = [
    {"ip": "192.168.1.10", "port": 54321, "role": "primary"},
    {"ip": "192.168.1.11", "port": 54321, "role": "standby"},
]

# 遍历节点
for node in cluster:
    print(f"连接 {node['role']} 节点：{node['ip']}:{node['port']}")

# 模拟主备切换：主节点故障，备节点接管
cluster[0]["role"] = "standby"   # 原主降为备
cluster[1]["role"] = "primary"   # 原备升为主

# 按角色找出当前主节点
primary = [n for n in cluster if n["role"] == "primary"][0]
print(primary["ip"])             # 192.168.1.11
```

> 对比：元组不可变、无法改 `role`；字典可改，适合角色会变化的场景。

### 4. 集合 set（无序、去重）
ids = {1, 2, 3, 3, 2}   # {1, 2, 3} 自动去重

---

## 四、控制流

### 条件判断

```python
score = 85

if score >= 90:
    print("优秀")
elif score >= 60:
    print("及格")
else:
    print("不及格")
```

### 循环

```python
# for 循环：遍历序列
for i in range(5):      # range(5) -> 0,1,2,3,4
    print(i)

# while 循环：条件成立时反复执行
count = 0
while count < 3:
    print(count)
    count += 1          # 注意：Python 没有 count++

# 循环控制
break     # 跳出整个循环
continue  # 跳过本次，继续下一次
```

---

## 五、函数

```python
def add(a, b=0):        # b=0 是默认参数
    """返回两数之和"""
    return a + b

add(1, 2)      # 3
add(1)         # 1，使用默认值
add(b=2, a=1)  # 3，关键字参数可以不按顺序
```

进阶概念（搭框架会用到）：

```python
# *args 接收任意个位置参数 -> 元组
def sum_all(*args):
    return sum(args)

sum_all(1, 2, 3, 4)     # 10

# **kwargs 接收任意个关键字参数 -> 字典
def show_info(**kwargs):
    for k, v in kwargs.items():
        print(f"{k}: {v}")

show_info(name="张三", age=25)
```

---

## 六、类与面向对象（测试框架核心）

自动化测试框架（pytest、unittest）大量使用类，必须理解。

```python
class TestCase:
    """一个测试用例类"""

    def __init__(self, name):   # 构造方法，self 代表实例本身
        self.name = name         # self.name 是实例属性

    def run(self):               # 实例方法，第一个参数永远是 self
        print(f"运行用例：{self.name}")


case = TestCase("登录测试")   # 创建实例，自动调用 __init__
case.run()                     # 调用方法 -> 运行用例：登录测试
```

关键点：

| 概念 | 说明 |
|------|------|
| `__init__` | 构造方法，创建实例时自动调用 |
| `self` | 代表实例本身，方法第一个参数必须是它 |
| 实例属性 | 用 `self.xxx` 定义，可在所有方法中访问 |
| 继承 | `class Child(Parent)` 继承父类，可重写方法 |

```python
# 继承示例
class ApiTest(TestCase):
    def run(self):          # 重写父类方法
        print(f"调用接口：{self.name}")
```

---

## 七、模块与导入

把代码拆到多个文件，通过 `import` 复用——这是框架组织的基石。

```python
# 导入整个模块
import os
os.getcwd()              # 获取当前目录

# 从模块导入特定函数/类
from datetime import datetime
now = datetime.now()

# 导入自定义模块（同目录下的 my_utils.py）
import my_utils
my_utils.login()
```

常用内置模块：

| 模块 | 用途 |
|------|------|
| `os` | 文件路径、目录操作、环境变量 |
| `json` | 读写 JSON 数据（接口测试必备） |
| `time` / `datetime` | 时间处理 |
| `random` | 生成随机数 |

```python
import json

data = {"name": "张三", "age": 25}
json_str = json.dumps(data)      # 字典 -> JSON 字符串
data2 = json.loads(json_str)     # JSON 字符串 -> 字典
```

---

## 八、异常处理（测试必备）

捕获异常可以防止脚本崩溃，并记录错误信息。

```python
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"出错了：{e}")
except Exception as e:      # 兜底捕获所有异常
    print(f"未知错误：{e}")
else:
    print("没有异常时执行")
finally:
    print("无论是否异常都会执行")
```

> 💡 测试框架中常用 `try/except` 捕获断言失败或接口异常，保证用例继续执行。

---

## 九、文件操作

```python
# 读取文件（推荐 with 写法，自动关闭）
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()          # 一次性读取
    # 或逐行读取
    # for line in f:
    #     print(line)

# 写入文件
with open("out.txt", "w", encoding="utf-8") as f:
    f.write("hello")
```

---

## 十、新手易错点

1. **缩进是语法**：用 4 个空格，别混用 Tab，缩进错误直接报错。
2. **没有 `++` / `--`**：用 `+= 1` 代替。
3. **字符串和数字不能直接拼接**：先 `str()` 转换，或用 f-string。
4. **`is` 和 `==` 不同**：`==` 比较值，`is` 比较是否同一对象。
5. **可变类型默认参数陷阱**：默认参数别用列表 `def f(a=[])`，用 `None` 代替。

---

## 下一步

掌握了以上基础语法，就可以进入下一阶段：

- **第二周**：`pytest` 测试框架（断言、fixture、参数化）
- **第三周**：接口自动化（`requests` + `json`）
- **第四周**：测试报告与 CI 集成

> 本文持续更新中，欢迎在评论区交流。
