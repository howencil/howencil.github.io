---
title: "Go语言初始化配置"
date: 2022-06-11T02:36:59+08:00
draft: false
author: 插画师
tags: ["Go"]
categories: ["Tech"]
---

# 1.使用homebrew安装go

```bash
brew install go
```

# 2.设置代理

```bash
go env -w GOPROXY=https://goproxy.cn,https://goproxy.io,direct
```

> 参考文章：[「2022 版」轻松搞定 Go 开发环境](https://polarisxu.studygolang.com/posts/go/2022-dev-env/)



