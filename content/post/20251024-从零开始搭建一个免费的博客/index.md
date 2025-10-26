---
title: 从零开始搭建一个免费的博客
description: 
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

## 大致步骤 (待细化)
1. 安装 Hugo
2. 使用 Hugo 初始化项目
3. 安装 Stack 主题
4. 在 Github 创建仓库
5. 初始化本地 Git 仓库
6. 暂存本地所以变更,并提交到本地仓库 git add. && git commit -m "first commit"
7. 推送至 Github仓库 git push -u origin main
8. 创建 Cloudflare Pages 应用
9.  链接 Github 仓库
10. 配置构建命令 git submodule update --init --recursive && hugo --minify