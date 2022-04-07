---
title: meet the problems of buiding the hexo blog 
comments: false
date: 2018-02-06 15:59:44
updated: 2022-02-28 09:19:40
description: 本文描述了自己搭建hexo所遇到的问题。

tags:
	- Hexo
categories:
	- Hexo
---

# 搭建hexo和github遇到问题
学习一样新事物，总是要经历磕磕碰碰，才能有所成长！脚踏实地，一步一个脚印，走好每一步，不要给自己留遗憾！

---

## 安装hexo问题
安装hexo我是在网上找教程安装的：
1. 安装node.js(去官网下载windows版的安装包) [node.js官网](https://nodejs.org)
2. 安装git  [git下载地址](https://git-scm.com/downloads/)
3. 申请github账号 [github](https://github.com/) 这里自己注册

设置user.name user.email:

``` bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
```
                 
4. 生成ssh密匙

```bash
ssh-keygen -t rsa -C "你的GitHub注册邮箱"
```
          
一路回车就OK了！此时，在用户文件夹下就会有一个新的文件夹 `.ssh`，里面有刚刚创建的ssh密钥文件 `id_rsa` 和 `id_rsa.pub`。上传公匙添加到github上。***记住是id_rsa.pub内容复制到github上***。

5. 安装 hexo 

```bash
npm install hexo-cli g
```

创建一个文件夹 `hexo`。

```bash
mkdir hexo 
```
初始化 `hexo` 文件夹。

```bash
hexo init hexo
```

安装hexo插件，都安装了吧!

```bash
npm install hexo-server --save
npm install hexo-admin --save
npm install hexo-generator-archive --save
npm install hexo-generator-feed --save
npm install hexo-generator-search --save
npm install hexo-generator-tag --save
npm install hexo-deployer-git --save
npm install hexo-generator-sitemap --save
npm install hexo-generator-category --save
```

6. 本地使用hexo

生成静态页面
``` bash
hexo generate
```


开启本地服务器(可以使用全名 hexo server)
```bash
hexo s 
```

在浏览器输入 **[http://localhost:4000/](http://localhost:4000/)** 就可以在本地访问 blog。

7. hexo 部署到 github 上
 
先在github上创建一个仓库名字：*github用户名.github.io*。这个创库的命名规则不能弄错了！！！！！！

修改博客配置文件(`hexo/_config.yml`)，在文件里找到下面内容并修改。

``` 
# Deployment 注释
## Docs: https://hexo.io/docs/deployment.html
deploy:
  # 类型
  type: git
  # 仓库
  repo: git@github.com:用户名/用户名.github.io.git
  # 分支
  branch: master
```

记得一定要修改成自己的用户名！！！！ 这种方式使用ssh链接（前提是你添加了公钥到 github）。

还有一种方式是把repo 修改为下面
```  bash
 repo: https://github.com/用户名/用户名.github.io
```
由于我用了两个 github 账号，建立 hexo 博客是换了一个新的账号，使用这种方式总是报 403 错误，我是改用 ssh 连接 github。

输入下面的命令将 hexo 博客部署到 github 中：
``` bash
# 清空静态页面
hexo clean
# 生成静态页面
hexo generate
# 部署 
hexo deploy
```
这里有一点建议：发布时最好连续执行这三个命令，以免部署到 github 的 hexo 为以前的，就相当于刷新一下缓存。

---

## hexo替换主题

我使用的next主题

1. 打开 git bash 并切换到你的博客目录下 比如我的博客目录是 `/hexo`
``` bash
#自己切换自己的博客目录
$ cd hexo
# 执行下面命令
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
2. 修改博客配置文件 `/hexo/_config.yml` 如下：
``` 
# Extensions
## Plugins: http://hexo.io/plugins/
## Themes: http://hexo.io/themes/
#以前的选项为     theme: landscape
theme: next
```
3. 修改主题配置文件 `/hexo/themes/next/_config.yml`

---

## menu 修改

找到下面内容

```
# ---------------------------------------------------------------
# Menu Settings
# ---------------------------------------------------------------
menu:
  home: / || home
  #about: /about/ || user
  tags: /tags/ || tags
  #categories: /categories/ || th
  #archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

去掉 # 就可以多一个菜单栏 比如去掉 

``` 
menu:
  home: / || home
  #about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  #archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

虽然在首页能看见这个菜单，但是是不能使用。点击菜单 categories 会返回 `can't get /categories/` 错误信息。

首先要安装categories对应插件

``` bash
$ npm install hexo-generator-category --save
```
再在gitbash输入（切换到 `hexo` 目录下）

``` bash
$ hexo new page categories
```
在 `/hexo/source` 目录下发现多的一个categories文件夹 进入并编辑 `index.md`。

``` 
---
title: categories
date: 2018-1-22 12:39:04
type: "categories"
comments: false
---
```

在 `/hexo/source/_post` 目录建立 `test.md` 内容如下：

``` 
---
title: test
categories:
  - Testing
---
```
注意空格！！！！注意空格！！！！注意空格！！！！注意空格！！！！

---

## 赞赏功能问题

在主题配置文件找到如下内容，并修改如下：

```bash
# Reward
reward_comment: Donate
wechatpay: /images/wechatpay.png
alipay: /images/alipay.jpg
#bitcoin: /images/bitcoin.png
```

自己图片位置在 `Hexo/themes/next/source` 目录下的 `image`文件内。修改成这样过后但是还是不能显示赞赏功能。至少我是这样，经过百度终于解决了。

---

在主题配置文件查找

```bash
# Automatically Excerpt. Not recommend.
# Please use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
  enable: false
  length: 150
```

修改为如下:

```bash
# Automatically Excerpt. Not recommend.
# Please use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
  enable: true
  length: 150
```
再执行
``` bash
$ hexo clean && hexo generate &&  hexo s 
```
这下应该可以！

---

## 本地搜索配置
1. hexo配置文件 `/hexo/_config.yml` 添加
```
#search
search:
  path: search.xml
  field: post
  format: html
  limit: 10000
```
2. 安装插件
``` bash
 $ npm install hexo-generator-search --save
 $ npm install hexo-generator-searchdb --save
```
3. 修改主题配置文件 `/hexo/themes/next/_config.yml`
```
# Local search
# Dependencies: https://github.com/flashlab/hexo-generator-search
local_search:
  enable: false
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
```
修改为
```
# Local search
# Dependencies: https://github.com/flashlab/hexo-generator-search
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
```


---


## 备注

I would greatly appreciate hearing about any error in this article, even minor ones.
I welcome your suggestions for improvements, even tiny one. Please email to me!😜 


---


## 参考文档
[next参考文档](http://theme-next.iissnan.com/)
[hexo参考文档](https://hexo.io/zh-cn/docs/)

没事就多看看官方文档，应该能解决问题的！加油！
