---
title: 使用hexo和github搭建个人blog
date: 2023-03-16 09:33:20
tags: 
    - "blog"
    - "hexo"
    - "github"  
---

这里记录下自己使用hexo，github 搭建自己的blog的一个过程，方便有需要的朋友可以参考.非小白教程，这里默认首先你有github帐号,并且能够使用git进行github仓库的基本操作。
主要操作流程就是：

1. 搭建一个基于hexo的blog项目，并提交特定的github仓库的主干分支

2. 定义github的Actions，提交代码会触发执行hexo 构建生成HTML到public目录，并执行发布github空间.

## 使用hexo框架搭建blog

参考官网: https://hexo.io/zh-cn/docs/
常用的几个命令：

```bash
hexo clean 或者 hexo c # 清理生成文件和缓存
hexo generate 或者 hexo g # 生成html文件到发布目录
hexo server 或者 hexo s # 启动本地服务
```

更多命令可以参考：https://hexo.io/zh-cn/docs/commands

## 发布github

1. github上创建一个仓库，名称为`你的github帐号.github.io`。
2. 进入仓库的`Settings` 配置tab页，选择侧边栏的 `Pages`。在 `Build and deployment` 选项下的 `Branch` 选择`gh-pages分支的/root`。
3. 进入仓库的`Actions` 配置tab页，创建一个workflows.
名称pages.yml，这里跟官方给的例子稍微不一样，node.js 16版本的，需要的插件版本需要升级到v3,以下这个版本是可以使用的。

```yml
name: Pages

on:
  push:
    branches:
      - main # default branch
jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js 16.x
        uses: actions/setup-node@v3
        with:
          node-version: "16.15.0" # 配置你的nodejs环境
      - name: Cache NPM dependencies
        uses: actions/cache@v3
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

## 使用自定义域名

默认github 发布给我们的地址是 `https://github帐号.github.io`,但是也支持使用我们自己的域名，而且支持https，会自动帮我们配置Let's Encrypt的免费证书，这点非常好。你要做的就是去购买域名，域名解析CNAME到 你的github.io地址就可以了。

1. 域名购买，域名解析（略...）
2. 打开github仓库的 `Settings`,在侧边栏 Pages 有个`Custom domain` 填入你的域名。如果你不希望提供http服务，只想提供https，记得`Enforce HTTPS` 选型。这样 http 会302 重定向到 https服务。
3. 等域名解析生效后访问。https 服务的证书更新则需要一段时间，现在打开会报证书无效(使用的github的证书)。

## 参考

- [hexo-theme-pure主题](https://github.com/cofess/hexo-theme-pure)
