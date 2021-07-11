# 搭建 hugo 博客部署到 firebase

## 前言

你需要对于 git 和 github 以及 firebase 有基本的了解，并且有对应的账号。我们将要把博客源码托管到 github，把生成到网站使用 firebase 的 hosting 来托管。

本文假定你是希望创建一个和我的博客一样的网站，然后进行创作，而无关网站的样式。

## 初始化

1. 将我的仓库 fork 到自己的 github 下。
2. 「可选」安装 hugo 的环境，便于在本地预览和部署，参见 hugo 官方 [Quick Start](https://gohugo.io/getting-started/quick-start/)

## 需要注意的点

### 新建文章

在 post 目录下新建 md 文件即可。你需要手动指定 title 和 date。

### 修改站点图标

修改 `static/favicon.ico` 文件即可。

## 部署到 firebase

### 创建 firebase 项目

在 firebase 的控制台中创建一个项目，记住项目的 id。修改 `.firebaserc` 中的 `default` 字段。

### 创建 firebase ci 配合 github action 实现自动提交

hugo 官方提供了很详细的[文档](https://gohugo.io/hosting-and-deployment/hosting-on-firebase/)，我们只需要最后 ci 的部分。

请注意 firebase-tools 需要 npm 环境，通过 nvm 来安装 node 并管理版本，node 自带 npm 环境。

输入 `firebase login:ci` ，浏览器中登录之后会得到一个 token，注意这里生成的 token 要只能自己知道。

这时我们已经可以通过 `hugo && firebase deploy --token <生成的 token>` 来部署到 firebase

在你的 github 项目的 `settings/secrets` 中添加一个名为 `FIREBASE_TOKEN` 的值，传入我们得到的 token。

这样配置完成之后，每次我们提交到 github 上面的时候，它就会自动为我们部署到 firebase 上面。
