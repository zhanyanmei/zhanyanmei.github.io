# /blog-build — Hexo 博客构建与维护

> 调用方式：`/blog-build` 或直接告诉 Claude "维护博客"、"更新博客"、"发新文章" 等

---

## 项目信息

| 项 | 值 |
| --- | --- |
| **本地路径** | `d:\其他\blog` |
| **GitHub 仓库** | `https://github.com/zhanyanmei/zhanyanmei.github.io` |
| **线上地址** | `https://zhanyanmei.github.io` |
| **主题** | Butterfly 5.6.1 |
| **部署方式** | GitHub Actions → `gh-pages` 分支 |

---

## 核心工作流

```
编辑文件 → hexo generate → git commit → git push → GitHub Actions → 30秒后上线
```

### 标准操作流程

```bash
# 1. 进入博客目录
cd "d:\其他\blog"

# 2. 编辑文件（写文章、改配置等）
# ...

# 3. 构建验证
npx hexo clean && npx hexo generate

# 4. 提交并推送
git add -A
git commit -m "描述你做了什么改动"
git push

# 5. 等 30 秒，检查部署状态
# 访问 https://zhanyanmei.github.io 确认
```

---

## 目录结构

```
d:\其他\blog/
├── _config.yml                  ← 站点主配置（URL、插件、SEO）
├── _config.butterfly.yml        ← Butterfly 主题配置（菜单、社交、侧边栏）
├── source/
│   ├── _posts/                  ← 所有文章（.md）
│   ├── about/index.md           ← 关于页面
│   ├── projects/index.md        ← 项目展示页面
│   ├── tags/index.md            ← 标签索引页
│   └── categories/index.md      ← 分类索引页
├── themes/
│   └── butterfly/               ← Butterfly 主题（npm 包）
├── scaffolds/                   ← 文章模板
├── .github/workflows/deploy.yml ← GitHub Actions 自动部署
├── .gitignore                   ← 忽略 node_modules/、public/
├── package.json                 ← 依赖声明
└── public/                      ← 构建产物（已 gitignore，由 CI 生成）
```

---

## 关键配置文件说明

### `_config.yml` — 站点配置

修改这些字段会影响全站：

| 字段 | 作用 | 示例值 |
| --- | --- | --- |
| `title` | 网站标题 | `我的博客` |
| `description` | SEO 描述 | `一个关于技术、学习与成长的博客` |
| `keywords` | SEO 关键词 | `技术, 编程, Python` |
| `theme` | 当前主题 | `butterfly` |
| `syntax_highlighter` | 代码高亮器 | `prismjs` |
| `permalink` | 文章 URL 格式 | `:abbrlink.html` |
| `per_page` | 每页文章数 | `12` |

### `_config.butterfly.yml` — 主题配置

| 字段 | 作用 |
| --- | --- |
| `menu` | 导航栏菜单项 |
| `social` | 侧边栏社交链接 |
| `aside` | 侧边栏卡片开关 |
| `local_search` | 本地搜索开关 |
| `post_copyright` | 文章版权声明 |
| `wordcount` | 字数统计 & 阅读时间 |

---

## 日常操作

### 写新文章

```bash
cd "d:\其他\blog"
hexo new "文章标题"
# 编辑 source/_posts/文章标题.md
```

文章 front-matter 模板：

```yaml
---
title: 文章标题
categories:
  - 编程
tags:
  - Python
  - Hexo
description: 文章摘要（可选，用于 SEO）
top: false           # 置顶文章设为 true
---
```

### 新建独立页面

```bash
mkdir -p "d:\其他\blog\source\页面名"
# 创建 source/页面名/index.md，内容：
---
title: 页面标题
type: page
comments: false
---
```

然后在 `_config.butterfly.yml` 的 `menu` 中添加导航项。

### 修改导航菜单

编辑 `_config.butterfly.yml` 的 `menu` 字段：

```yaml
menu:
  首页: / || fas fa-home
  归档: /archives/ || fas fa-archive
  分类: /categories/ || fas fa-folder-open
  标签: /tags/ || fas fa-tags
  项目: /projects/ || fas fa-code
  关于: /about/ || fas fa-user
```

格式：`显示名: 路径 || 图标类名`

### 添加/修改社交链接

编辑 `_config.butterfly.yml` 的 `social` 字段：

```yaml
social:
  fas fa-envelope: mailto:你的邮箱 || 邮箱
  fab fa-github: https://github.com/你的用户名 || GitHub
```

---

## 已安装的插件

| 插件 | 用途 |
| --- | --- |
| `hexo-generator-sitemap` | 生成 sitemap.xml（SEO） |
| `hexo-generator-feed` | 生成 RSS/Atom 订阅 |
| `hexo-generator-searchdb` | 生成本地搜索数据 |
| `hexo-word-counter` | 字数统计 & 阅读时间 |
| `hexo-abbrlink` | 短链接（文章 URL 用 crc32 加密） |

安装新插件：

```bash
cd "d:\其他\blog"
npm install 插件名 --save
# 然后在 _config.yml 中添加对应配置
```

---

## 部署流程（GitHub Actions）

推送 `main` 分支后，`.github/workflows/deploy.yml` 自动执行：

```yaml
checkout → npm ci → hexo generate → deploy to gh-pages
```

- `main` 分支：源代码（_config.yml、source/_posts/...）
- `gh-pages` 分支：构建产物（public/ 目录，由 Actions 自动推送）

### 检查部署状态

```bash
curl -s https://api.github.com/repos/zhanyanmei/zhanyanmei.github.io/actions/runs | python -c "
import sys,json
data=json.load(sys.stdin)
for r in data['workflow_runs'][:3]:
    print(f'{r[\"name\"]:30s} {r[\"status\"]:12s} {r[\"conclusion\"]}')
"
```

### 如果 gh-pages 分支不存在

检查 Actions 日志中 "Deploy to GitHub Pages" 步骤是否成功。权限配置：仓库 Settings → Actions → General → Workflow permissions → **Read and write permissions**。

---

## 故障排查

### 构建失败

```bash
cd "d:\其他\blog"
npx hexo clean && npx hexo generate
# 根据报错信息定位问题
```

常见原因：
- `_config.yml` 或 `_config.butterfly.yml` YAML 语法错误（重复 key、缩进不对）
- 插件版本不兼容 → `npm update`

### 标签/分类页面空白

确认：
1. 文章 front-matter 中有 `tags: [...]` 或 `categories: [...]`
2. 存在 `source/tags/index.md`（`type: tags`）
3. 存在 `source/categories/index.md`（`type: categories`）

### 线上不更新

1. 检查 Actions 是否成功：https://github.com/zhanyanmei/zhanyanmei.github.io/actions
2. 检查 Pages 设置：https://github.com/zhanyanmei/zhanyanmei.github.io/settings/pages — Branch 选 `gh-pages`
3. 清除浏览器缓存 `Ctrl+Shift+R`

---

## 相关资源

- Hexo 文档：https://hexo.io/zh-cn/docs/
- Butterfly 主题文档：https://butterfly.js.org/
- Font Awesome 图标库：https://fontawesome.com/icons
- 本项目的 Hexo 搭建指南：`d:\其他\project_auto\test\doc\Hexo博客搭建指南.md`
