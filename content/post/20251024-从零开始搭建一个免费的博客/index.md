---
title: 从零开始搭建一个免费的博客
description: 
draft: false
date: 2025-10-24
slug: build-a-free-blog
image: https://ryan1998.dpdns.org/cover/post/build/build-a-free-blog.png
categories:
    - Cloudflare
    - Github
    - Hugo
tags:
    - Cloudflare Pages
    - Cloudflare R2
    - Hugo
---

# 基于Hugo + GitHub + Cloudflare Pages的博客搭建 - 图形化指南

## 🎯 整体架构图

```mermaid
graph TB
    A[👨‍💻 开发者] --> B[📝 本地Hugo项目]
    B --> C[🔄 Git推送]
    C --> D[🚀 GitHub仓库]
    D --> E[🌐 Cloudflare Pages]
    E --> F[⚙️ 自动构建]
    F --> G[📄 生成静态网站]
    G --> H[🚀 生产环境部署]
    I[👥 访客] --> H
    
    style A fill:#e1f5fe
    style B fill:#f3e5f5
    style D fill:#fff3e0
    style E fill:#e8f5e8
    style H fill:#4caf50
```

## 1. 📋 环境准备流程图

```mermaid
graph TB
    A[😀用户] --> B[注册Cloudflare账号]
    B --> C[添加支付方式]
    C --> D[✔准备完成]
    
    A --> E[注册Github账号]
    E --> F[创建Github仓库]
    F --> D
    
    A --> G[安装Git]
    G --> H[安装Hugo]
    H --> D
    
    
    style A fill:#e0f2f1
    style B fill:#f3e5f5
    style C fill:#fff3e0
    style D fill:#e8f5e8
    style E fill:#fce4ec
    style F fill:#e1f5fe 
```

## 2. 🚀 项目初始化流程

### 2.1 Hugo项目创建

```mermaid
flowchart TD
    A[hugo new site] --> B[创建项目结构]
    B --> C[安装主题]
    C --> D[配置config.toml]
    D --> E[本地测试]
```

```shell
终端命令执行界面
┌─────────────────────────────────────────────────┐
│ $ hugo new site my-blog                         │
│ Congratulations! Your new Hugo site is created! │
│                                                 │
│ $ cd my-blog                                    │
│ $ git init                                      │
│ Initialized empty Git repository...             │
│                                                 │
│ $ cd themes                                     │
│ $ git submodule add https://github.com/themes/  │
│ anatole.git                                     │
└─────────────────────────────────────────────────┘
```

### 2.2 项目目录结构

```
my-blog/
├── 📁 archetypes/          # 内容模板
├── 📁 content/             # 📝 文章内容
│   └── 📁 posts/           # 博客文章
├── 📁 data/                # 网站数据
├── 📁 layouts/             # 布局文件
├── 📁 static/              # 🖼️ 静态资源
│   ├── 📁 images/
│   └── 📁 css/
├── 📁 themes/              # 🎨 主题文件
│   └── 📁 anatole/         # 选择的主题
├── 📄 config.toml          # ⚙️ 配置文件
└── 📄 README.md            # 项目说明
```

## 3. ⚙️ Hugo配置界面

### 3.1 config.toml 配置文件

```yaml
# 🎯 网站基本信息
baseurl: https://ryan1998.pages.dev/
languageCode: en-us
theme: stack
title: Ryan's Blog
copyright: Ryan Young
DefaultContentLanguage: zh-cn
hasCJKLanguage: true

languages:
  zh-cn:
    languageName: 中文
    title: Ryan's Blog
    weight: 2
    params:
      sidebar:
        subtitle: 思考 | 学习 | 分享

services:
  # Change it to your Disqus shortname before using
  disqus:
    shortname: "stack"
  # GA Tracking ID
  googleAnalytics:
    id:

pagination:
  pagerSize: 10
  
permalinks:
  post: /p/:slug/
  page: /:slug/

params:
  mainSections:
    - post
  featuredImageField: image
  rssFullContent: true
  favicon: /favicon.ico # /static/favicon.ico
  
  footer:
    since: 2025
    customText: 持续学习，共同进步！ 🚀

  dateFormat:
    published: 2006-01-02 # Jan 02, 2006
    lastUpdated: Jan 02, 2006 15:04 MST

  sidebar:
    # emoji: 🤔
    subtitle: Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    avatar:
      enabled: true
      local: true
      src: img/think-emoji.png # /assets/img/think-light.png

  article:
    math: false
    toc: true
    readingTime: true
    license:
      enabled: true
      default: Licensed under CC BY-NC-SA 4.0

  comments:
    enabled: true
    provider: giscus

    giscus:
      repo: "ryanyoung1998/blog"
      repoID: "xxxxxxxx"
      category: "Announcements"
      categoryID: "xxxxxxxx"
      mapping: "pathname"
      strict: "0"
      reactionsEnabled: "1"
      emitMetadata: "0"
      inputPosition: "bottom"
      lightTheme: "light"
      darkTheme: "dark"
      lang: "zh-CN"
      crossorigin: "anonymous"

  widgets:
    homepage:
      - type: search
      - type: archives
        params:
          limit: 5
      - type: categories
        params:
          limit: 10
      - type: tag-cloud
        params:
          limit: 10
    page:
      - type: toc

  opengraph:
    twitter:
      # Your Twitter username
      site:
      # Available values: summary, summary_large_image
      card: summary_large_image

  defaultImage:
    opengraph:
      enabled: false
      local: false
      src:

  colorScheme:
    # Display toggle
    toggle: true

    # Available values: auto, light, dark
    default: auto

  imageProcessing:
    cover:
      enabled: true
    content:
      enabled: true

menu:
  main: []

  social:
    - identifier: github
      name: GitHub
      url: https://github.com/ryanyoung1998/blog
      params:
        icon: github-icon # /assets/img/github-icon.svg
  
    - identifier: bilibili
      name: Bilibili
      url: https://space.bilibili.com/27682859
      params:
        icon: bilibili-icon # /assets/img/bilibili-icon.svg

related:
  includeNewer: true
  threshold: 60
  toLower: false
  indices:
    - name: tags
      weight: 100
    - name: categories
      weight: 200

markup:
  goldmark:
    extensions:
      passthrough:
        enable: true
        delimiters:
          block:
            - - \[
              - \]
            - - $$
              - $$
          inline
            - - \(
              - \)
    renderer
      ## Set to true if you have HTML content inside Markdown
      unsafe: true
  tableOfContents:
    endLevel: 4
    ordered: true
    startLevel: 2
  highlight:
    noClasses: false
    codeFences: true
    guessSyntax: true
    lineNoStart: 1
    lineNos: true
    lineNumbersInTable: true
    tabWidth: 4
```

## 4. 📝 内容创作流程

### 4.1 新建文章界面

```shell
终端命令
┌─────────────────────────────────────────────────┐
│ $ hugo new posts/my-first-post.md               │
│ Content created at content/posts/my-first-post.md│
└─────────────────────────────────────────────────┘
```

### 4.2 文章编辑器界面

```markdown
---
title: "我的第一篇博客文章"
date: 2024-01-15T10:00:00+08:00
draft: false
tags: ["Hugo", "教程"]
categories: ["技术"]
summary: "这是我的第一篇使用Hugo搭建的博客文章"
---

## 🎉 欢迎阅读

这是我的第一篇博客文章，使用Hugo静态网站生成器创建。

## 📝 代码示例

```python
def hello_world():
    print("Hello, Hugo!")
    return "欢迎来到我的博客"
```

## 🖼️ 图片插入

![示例图片](/images/sample.jpg)

## 🔗 相关链接

- [Hugo官方文档](https://gohugo.io/)
- [GitHub仓库](https://github.com/your-username/my-blog)


## 5. 🔧 本地开发调试

### 5.1 本地服务器启动

```mermaid
sequenceDiagram
    participant U as 用户
    participant H as Hugo服务
    participant B as 浏览器

    U->>H: hugo server -D
    H->>H: 启动开发服务器
    H-->>U: 服务器运行在 http://localhost:1313
    U->>B: 打开浏览器访问
    B->>H: 请求页面
    H-->>B: 返回渲染页面
    U->>H: 修改文件自动重载
    H-->>B: 自动刷新页面
```

```shell
本地开发终端界面
┌─────────────────────────────────────────────────┐
│ $ hugo server -D --navigateToChanged            │
│                                                 │
│ Start building sites ...                        │
│ hugo v0.120.4+extended ... buildDate: unknown   │
│ Running in Fast Render Mode. For full rebuilds  │
│ on change: hugo server --disableFastRender      │
│ Web Server is available at http://localhost:1313│
│ Press Ctrl+C to stop                            │
│                                                 │
│ Change detected, rebuilding site.               │
│ Total in 45 ms                                  │
└─────────────────────────────────────────────────┘
```

## 6. 📤 推送到GitHub

### 6.1 Git仓库配置

```shell
终端配置界面
┌─────────────────────────────────────────────────┐
│ # 配置Git用户信息                              │
│ $ git config --global user.name "Your Name"     │
│ $ git config --global user.email "your-email@example.com"│
│                                                 │
│ # 初始化Git仓库                                │
│ $ git init                                      │
│ $ git add .                                     │
│ $ git commit -m "初始提交: Hugo博客项目"        │
│                                                 │
│ # 连接到GitHub远程仓库                         │
│ $ git remote add origin https://github.com/your-│
│ username/my-blog.git                            │
│ $ git branch -M main                            │
│ $ git push -u origin main                       │
└─────────────────────────────────────────────────┘
```

## 7. 🌐 Cloudflare Pages配置

### 7.1 连接GitHub仓库

```mermaid
flowchart TD
    A[登录Cloudflare] --> B[进入Pages]
    B --> C[创建项目]
    C --> D[连接GitHub账号]
    D --> E[选择仓库]
    E --> F[配置构建设置]
    F --> G[开始部署]
```

## 8. 🔄 自动化构建流程

### 8.1 构建部署流程图

```mermaid
sequenceDiagram
    participant D as 开发者
    participant G as GitHub
    participant C as Cloudflare Pages
    participant B as 构建环境
    participant CDN as 全球CDN

    D->>G: git push origin main
    G->>C: Webhook触发构建
    C->>B: 启动构建容器
    B->>B: git clone仓库
    B->>B: hugo构建静态网站
    B->>B: 生成public目录
    B->>CDN: 部署到全球网络
    C->>D: 构建成功通知
```

## 10. 🔄 日常维护流程

### 10.1 内容更新工作流

```mermaid
flowchart TD
    A[撰写新文章] --> B[本地测试]
    B --> C[推送到GitHub]
    C --> D[自动部署]
    D --> E[在线发布]
```

### 10.2 快速更新命令

```shell
日常更新终端界面
┌─────────────────────────────────────────────────┐
│ # 1. 创建新文章                                │
│ $ hugo new posts/$(date +%Y-%m-%d)-new-post.md  │
│                                                 │
│ # 2. 本地预览                                  │
│ $ hugo server -D                                │
│                                                 │
│ # 3. 发布到生产环境                            │
│ $ git add .                                     │
│ $ git commit -m "发布新文章: $(date +%Y-%m-%d)" │
│ $ git push origin main                          │
│                                                 │
│ # 4. 等待自动部署完成                          │
│ # 访问: https://my-blog.pages.dev              │
└─────────────────────────────────────────────────┘
```
