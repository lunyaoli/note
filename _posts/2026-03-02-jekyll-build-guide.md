---
title: Jekyll Theme Chirpy 编译启动流程
description: >-
  详细介绍 Jekyll Theme Chirpy 的编译和启动流程，包括前端资源构建、Jekyll 构建、服务启动，以及常见问题的解决方案。
author: lunyaoli
date: 2026-03-02 18:06:00 +0800
categories: [Blogging, Tutorial]
tags: [jekyll, chirpy, build]
---

## 前端资源构建

```bash
npm install        # 安装 Node.js 依赖
npm run build      # 构建 CSS + JS
```

`npm run build` 会并发执行：
- `build:css` - 运行 purgecss 优化 CSS
- `build:js` - 用 rollup 打包 JS 文件到 `assets/js/dist/`

### 输出的 JS 文件

| 文件 | 用途 |
|-----|-----|
| `theme.min.js` | 主题切换（深色/浅色） |
| `home.min.js` | 首页功能 |
| `post.min.js` | 文章页面功能 |
| `page.min.js` | 普通页面功能 |
| `categories.min.js` | 分类页功能 |
| `commons.min.js` | 公共模块 |
| `misc.min.js` | 其他功能 |
| `app.min.js` | PWA 应用 |
| `sw.min.js` | Service Worker |

## Jekyll 构建

```bash
bundle exec jekyll build
```

将源文件编译到 `_site/` 目录，包含：
- HTML 页面
- CSS 样式
- JS 脚本
- 图片等静态资源

## 启动服务

```bash
bundle exec jekyll serve --detach --host 0.0.0.0 --port 4000
```

参数说明：
- `--detach` - 后台运行（守护进程模式）
- `--host 0.0.0.0` - 允许外部访问（不只是 localhost）
- `--port 4000` - 指定端口（默认 4000）

## 常见问题与解决方案

| 问题 | 原因 | 解决方法 |
|-----|-----|---------|
| `Address already in use` | 端口被占用 | `lsof -ti:4000 \| xargs kill -9` |
| JS 文件 404 | 未构建前端资源 | `npm run build` |
| `_site/` 目录缺少资源 | 未运行 jekyll build | `bundle exec jekyll build` |
| 样式丢失 | CSS 未构建 | `npm run build` |

## 完整重启流程

```bash
# 进入项目目录
cd /root/.openclaw/workspace/jekyll-theme-chirpy

# 杀掉占用端口的进程
lsof -ti:4000 | xargs kill -9 2>/dev/null

# 构建前端资源
npm run build

# 构建 Jekyll 站点
bundle exec jekyll build

# 启动服务（后台运行）
bundle exec jekyll serve --detach --host 0.0.0.0 --port 4000
```

## 开发模式（实时预览）

如果需要修改内容并实时预览：

```bash
# 前台运行，支持热重载
bundle exec jekyll serve --host 0.0.0.0 --port 4000 --livereload
```

## 停止服务

```bash
# 方法一：通过 PID
kill -9 <PID>

# 方法二：杀掉所有 jekyll 进程
pkill -f jekyll

# 方法三：通过端口
lsof -ti:4000 | xargs kill -9
```

## 项目结构

```
jekyll-theme-chirpy/
├── _config.yml          # Jekyll 配置
├── _posts/              # 博客文章
├── _tabs/               # 页面（About, Archives 等）
├── _data/               # 数据文件
├── _includes/           # 模板片段
├── _layouts/            # 页面布局
├── _sass/               # SCSS 样式源码
├── _javascript/         # JS 源码
├── assets/              # 静态资源
│   ├── js/dist/         # 编译后的 JS（npm run build 生成）
│   └── img/             # 图片
├── _site/               # 编译输出目录（jekyll build 生成）
├── Gemfile              # Ruby 依赖
└── package.json         # Node.js 依赖
```

## 依赖要求

- **Ruby**: 3.1.0+
- **Node.js**: 16+
- **Bundler**: `gem install bundler`
- **Jekyll**: 4.4.1
