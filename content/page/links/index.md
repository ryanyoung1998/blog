---
title: 链接
date: 2025-10-20
slug: "links"
links:
  - title: GitHub
    description: GitHub is the world's largest software development platform.
    website: https://github.com
    image: github.svg # https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png
  - title: Golang
    description: Build simple, secure, scalable systems with Go.
    website: https://golang.google.cn/
    image: golang.svg
menu:
    main:
        weight: -50
        params:
            icon: link

comments: false
---


要想使用链接功能？

需要在前言(frontmatter)部分添加`links`块。

`image` 可以填写本地URI和外部URL。

此页面的前言配置如下:

```yaml
links:
  - title: GitHub
    description: GitHub is the world's largest software development platform.
    website: https://github.com
    image: github.svg # https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png
  - title: Golang
    description: Build simple, secure, scalable systems with Go.
    website: https://golang.google.cn/
    image: golang.svg
```
