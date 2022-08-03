---
title: "Hugo搭建博客"
date: 2022-08-04T00:58:00+08:00
draft: false
tag:
- hugo
---

## 安装

以Ubuntu20.04为例


```
sudo apt-get install hugo
```

但是apt仓库版本一般不是最新的. 如果想安装最新版本, 可以从hugo的[GitHub发行仓库](https://github.com/gohugoio/hugo/releases)上获取deb包并安装. 例如:


```
# 进入用户目录, 随意.
cd ~
# 获取deb文件, 自行选择合适的版本.
wget https://github.com/gohugoio/hugo/releases/download/v0.92.2/hugo_0.92.2_Linux-64bit.deb
# 安装
sudo dpkg -i hugo*.deb
# 查看hugo版本
hugo version
```

## 配置

### 主题

#### 安装主题

```
git submodule add git@github.com:adityatelange/hugo-PaperMod.git themes/hugo-PaperMod
```
[*git submodule命令详解*](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

#### 使用主题

在配置文件```config.yaml```中使用如下配置项
```
theme: hugo-PaperMod
```

> 注意:  
> 如果重新git clone项目, 通过git submodule安装的主题, 并不会第一时间安装. 需要执行以下两条命令以安装主题.
> ```
> git submodule init
> git submodule update
> ```

## 基本操作

生成新文章
```
hugo new post/<filename>.md
```

为了简化操作, 在```.bashrc```文件里设置h函数
```
h(){
    hugo new post/$1.md
}
```

## 部署到GitHub Page

### 生成token

我自己笔记的源码和github.io的仓库分开, 为了能够获取仓库的读写权限, 需要生成personal token. 并在源码仓库里设置(token的名字要与workflow配置文件里一致).

### 设置workflow文件

生成```.github/workflows/gh-pages.yml```文件, 内容如下.

```
name: GitHub Pages

on:
  push:
    branches:
      - main  # Set a branch to deploy
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-20.04
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.92.2'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        if: ${{ github.ref == 'refs/heads/main' }}
        with:
          external_repository: KEAE12315/blog
          personal_token: ${{ secrets.ACTION_ACCESS_TOKEN }} 
          publish_dir: ./public
          publish_branch: hugo
```

> 参考资料  
> https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-first-deployment-with-github_token  
> [Hugo使用Github Action自动部署博客到Github Pages](https://tomial.github.io/posts/hugo%E4%BD%BF%E7%94%A8github-action%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%E5%8D%9A%E5%AE%A2%E5%88%B0github-pages/)
