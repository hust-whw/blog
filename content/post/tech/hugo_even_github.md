---
title: "Github Pages + Hugo 静态博客部署"
date: 2020-02-12
# lastmod: 2020-01-01
draft: false
tags: ['Hugo']
categories: ['技术文档']
# slug: 
---
<!--more-->
## 前言
很久之前就有搭建个人博客的想法，不过拖延症犯起来永无尽头，一直在用[notion](https://www.notion.so/)作为大脑管理工具，近期才想付诸于实践。

在对比Hugo，hexo和jekyll后，由于个人对golang有强烈兴趣（对比hexo的js和jekyll的ruby），并且还有众多golang带来的特性，毅然决然的选择了hugo作为静态博客生成器，整个流程简单高效，非常适合内容至上的博主。

本文会简单介绍我在本地安装配置hugo，使用Even作为hugo的主题，并将自动生成的静态网站部署至github pages的简单流程，还有一些个性化修改的tips。

## 安装

hugo官方有一个非常全面的安装流程，可以根据自己的操作系统进行选择，这里我选择用macOS自带的homebrew安装。如果本地没有安装homebrew，[戳这里](https://brew.sh/index_zh-cn)。

```shell
brew install hugo
```
查看版本号以验证安装成功
```shell
hugo version
```
## 新建站点

### 生成项目文件夹
```shell
# 会在当前目录建立文件名为blog的hugo项目
hugo new site blog
cd blog
tree .
```
（非必须）通过tree命令(`brew install tree`)可以打印新创建的hugo项目的树形结构。

### 安装主题
hugo没有默认主题，项目下的`theme`文件夹是空的，即安装主题这一步是必须的，否则hugo运行不起来。可以在[官方主题库](https://themes.gohugo.io/)选择喜欢的主题，我用的是从even主题。
```shell
# 在blog项目文件夹下
git init
git submodule add https://github.com/creaink/hugo-theme-even themes/even
```

### 配置
even主题推荐将`themes/even/exampleSite/config.toml`覆盖项目目录下`./config.toml`，然后再做修改。
不同主题的`config.toml`配置有所不同，even在配置文件中有详细的注释。
```shell
# 在blog项目文件夹下
cp themes/even/exampleSite/config.toml ./config.toml
```

（非必须）也可将`themes/even/exampleSite/content/`文件夹复制到项目下作为demo查看。
```shell
cp -r themes/even/exampleSite/content/ ./content/
```

自定义网站的favicon图标，需要生成各个尺寸的图标（使用[生成器1](https://www.favicon-generator.org/)或[生成器2(支持透明)](https://realfavicongenerator.net/)可以快速生成），生成后储存在`./static`文件夹下，用来覆盖`themes/even/static/`文件夹下主题中默认的图标。

### 尝试创建文章
```shell
hugo new post/hello-world.md
```

### 本地调试
```shell
hugo server
```
启动hugo本地服务器，默认监听1313端口，跑起来以后可以访问`http://localhost:1313/`查看页面进行调试或写作。

期间所有对文件进行的修改，hugo默认会自动应用到页面上，关闭此功能需要传参`hugo server -w false`，更多参数[参考这里](https://gohugo.io/commands/hugo_server/)。

### 生成静态页面
```shell
hugo
```
运行完命令，hugo会在项目下生成`public`文件夹，即为部署为网页所有的静态文件。

如想自定义文件夹名，需要在`config.toml`文件中添加一行配置`publishDir = "docs"`。

我这里设置成docs是因为，接下来我需要将项目通过github pages托管，而github pages有三种方式部署静态网页，我想在同一个仓库下管理源文件和静态网站，所以我选择的是将仓库下`docs`文件夹发布为网站的方式，下文会介绍。[了解更多](https://help.github.com/en/github/working-with-github-pages)

## 部署至github pages

### 上传至仓库
在自己的Github中添加一个空白repository，不用勾选README，.gitignore等选项。复制repo的地址，如`https://github.com/user-name/repo-name.git`

然后回到本地项目下
```shell
# 确保运行过`hugo`命令生成了`docs`文件夹。
hugo
# 由于刚开始运行了git init，这里不再运行
git add .
git commit -m "Initial Site"
# 发布到刚才建立的新repo
git remote add origin https://github.com/user-name/repo-name.git
git push -u origin master
```

这时github仓库已经托管了此hugo项目，包括生成好的网页静态文件。

### 配置github pages

在仓库repo的setting页面，找到GitHub Pages的设置块，将其中的source选择为`master branch /docs folder`。

这时候github pages已经将docs文件夹发布到`user-name.github.io`网站上，这里的user-name是github的UserName，可以访问该网址浏览。

勾选下方`Enforce HTTPS`选项会将网址变为有小绿锁的安全网站，推荐。配置后访问网站`https://user-name.github.io`。

### 配置自定义域名

如果觉得xxx.github.io太丑，或者觉得用别人的域名不爽，那么需要配置自己的个性域名。这里直接跳过怎么购买域名，默认拥有域名`example.com`。
这里需要分为两步，
####  **想要将页面部署在*主域名*下，即访问`example.com`就访问到页面**
1. 需要在Github Pages设置块中的Custom domain里填写自己的域名`example.com`
2. 在购买域名的服务商设置里添加CNAME，host设置为`@`，value设置为`user-name.github.io.`
3. 项目目录下添加CNAME文件，里面写自己的域名`example.com`。
4. 现在访问example.com就可以访问到博客页面
5. 勾选`Enforce HTTPS`选项，如不能勾选并提示`Not yet available for your site because the certificate has not finished being issued`,说明github在帮你的域名申请免费的SSL证书，需要等几个小时后可勾选。（如出现其他提示可尝试清空Custom domain，保存，再填写，再保存）
6. 这时候在设置页可以看见`Your site is published at https://example.com/`，大功告成。

#### **如果将页面部署在*子域名*下，即访问`blog.example.com`访问到页面**
1. 需要在Github Pages设置块中的Custom domain里填写自己的域名`blog.example.com`
2. 在购买域名的服务商设置里添加CNAME，host设置为`blog`，value设置为`user-name.github.io.`
3. 项目目录下添加CNAME文件，里面写自己的域名`blog.example.com`。如[我的设置](https://github.com/hust-whw/blog/blob/master/CNAME)
4. 现在访问blog.example.com就可以访问到博客页面
5. 勾选`Enforce HTTPS`选项，如不能勾选并提示`Not yet available for your site because the certificate has not finished being issued`,说明github在帮你的域名申请免费的SSL证书，需要等几个小时后可勾选。（如出现其他提示可尝试清空Custom domain，保存，再填写，再保存）
6. 这时候在设置页可以看见`Your site is published at https://blog.example.com/`，大功告成。

## 参考
- [Hugo官网](https://gohugo.io/)
- [Hugo官方文档](https://gohugo.io/documentation/)
- [Even主题github仓库](https://github.com/olOwOlo/hugo-theme-even)
- [Hugo 从入门到会用](https://blog.olowolo.com/post/hugo-quick-start)