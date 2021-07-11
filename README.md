# 搭建 hugo 博客部署到 firebase

## 前言

你需要对于 git 和 github 有基本的了解，并且有对应的账号。我们将要把博客源码托管到 github，把生成的网站使用 github pages 来托管。

本文假定你是希望创建一个和我的博客一样的网站，然后进行创作，而无关网站的样式。

## 初始化

1. 将我的仓库 fork 到自己的 github 下。
2. 「可选」安装 hugo 的环境，便于在本地预览和部署，参见 hugo 官方 [Quick Start](https://gohugo.io/getting-started/quick-start/)

## 需要注意的点

### 新建文章

在 post 目录下新建 md 文件即可。你需要手动指定 title 和 date。

### 修改站点图标

修改 `static/favicon.ico` 文件即可。
