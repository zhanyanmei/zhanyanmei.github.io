# 我的博客

基于 [Hexo](https://hexo.io/) 搭建的个人技术博客，使用 [Butterfly](https://butterfly.js.org/) 主题，通过 GitHub Actions 自动部署到 GitHub Pages。

**线上地址：** <https://zhanyanmei.github.io>

---

## 博客介绍

这是一个关于技术、学习与成长的个人博客，主要分享以下内容：

- **编程** — Python、自动化脚本、开发工具
- **自动化测试** — 测试框架、CI/CD、质量保障
- **数据库** — SQL、数据库设计与优化
- **随笔** — 学习笔记与生活记录

### 技术栈

| 组件 | 说明 |
|------|------|
| **框架** | Hexo 7.x |
| **主题** | Butterfly 5.6.1 |
| **代码高亮** | Prism.js |
| **部署** | GitHub Actions → gh-pages 分支 |
| **评论** | 待接入 |

### 已启用功能

- 本地搜索（hexo-generator-searchdb）
- RSS 订阅（hexo-generator-feed）
- 站点地图 SEO（hexo-generator-sitemap）
- 字数统计 & 阅读时间（hexo-word-counter）
- 短链接 permalink（hexo-abbrlink）
- Prism.js 代码高亮、代码行号

---

## 写文章并发布的流程

### 1. 创建新文章

```bash
cd "d:\其他\blog"
npx hexo new "你的文章标题"
```

这会在 `source/_posts/` 下生成 `你的文章标题.md`。

### 2. 编辑文章

打开生成的 `.md` 文件，修改 front-matter 并撰写正文：

```yaml
---
title: 文章标题
categories:
  - 编程          # 分类（可选：编程、数据库、测试、随笔）
tags:
  - Python        # 标签，可以有多个
  - Hexo
description: 文章摘要，用于 SEO 和列表展示（可选）
top: false        # 设为 true 可将文章置顶
---
正文内容（Markdown 格式）……
```

> **提示**：图片可以放在文章同级资源文件夹中（`post_asset_folder: true` 已开启），新建文章时会自动创建同名文件夹。

### 3. 本地预览（可选）

```bash
npx hexo server
```

浏览器打开 `http://localhost:4000` 查看效果，文件修改后自动刷新。

### 4. 构建验证

```bash
npx hexo clean && npx hexo generate
```

确认无报错，`public/` 目录生成了静态文件。

### 5. 提交并推送

```bash
git add -A
git commit -m "新增文章：你的文章标题"
git push
```

### 6. 等待自动部署

推送后 GitHub Actions 自动执行：`checkout → npm ci → hexo generate → deploy to gh-pages`

约 **30 秒**后访问 <https://zhanyanmei.github.io> 即可看到更新。

> 检查部署状态：<https://github.com/zhanyanmei/zhanyanmei.github.io/actions>

---

## 常用操作速查

### 文章相关

```bash
# 新建文章
npx hexo new "文章标题"

# 新建草稿（不会发布）
npx hexo new draft "草稿标题"

# 发布草稿
npx hexo publish "草稿标题"
```

### 页面相关

```bash
# 新建独立页面
mkdir -p "d:\其他\blog\source\页面名"
# 创建 source/页面名/index.md，front-matter 中写 type: page
```

### 导航菜单

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

格式：`显示名: 路径 || FontAwesome 图标类名`

### 站点配置

编辑 `_config.yml` 可修改：

| 配置项 | 作用 |
|--------|------|
| `title` | 网站标题 |
| `subtitle` | 副标题 |
| `description` | SEO 描述 |
| `keywords` | SEO 关键词 |
| `per_page` | 每页显示文章数 |

### 安装新插件

```bash
npm install 插件名 --save
# 然后在 _config.yml 中添加对应配置
```

---

## 目录结构

```text
d:\其他\blog/
├── _config.yml                  # 站点主配置
├── _config.butterfly.yml        # 主题配置（菜单、社交、侧边栏）
├── package.json                 # 依赖声明
├── source/
│   ├── _posts/                  # 所有文章
│   ├── about/index.md           # 关于页面
│   ├── projects/index.md        # 项目展示页面
│   ├── tags/index.md            # 标签索引页
│   └── categories/index.md      # 分类索引页
├── themes/butterfly/            # Butterfly 主题
├── scaffolds/                   # 文章模板
├── .github/workflows/deploy.yml # 自动部署脚本
└── public/                      # 构建产物（仅 CI 生成）
```

---

## 分支说明

| 分支 | 内容 |
|------|------|
| `main` | 博客源码（配置、文章、主题） |
| `gh-pages` | 构建产物（由 GitHub Actions 自动推送） |

---

## 相关链接

- [Hexo 官方文档](https://hexo.io/zh-cn/docs/)
- [Butterfly 主题文档](https://butterfly.js.org/)
- [Font Awesome 图标库](https://fontawesome.com/icons)
