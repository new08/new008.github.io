---
layout: post
title:  "搭建 jekyll"
date:   2017-08-24 17:49
categories: jekyll
---

#搭建 jekyll#

**环境** ubuntu 17

jekyll 是一个静态博客服务程序。支持使用 md 文本写博客。

###同类工具
hexo，基于 node 开发的博客程序。但是我在 ECS 上一直安装失败。

###安装 jekyll

a. 首先需要安装 ruby
    
``` 
sudo apt-get update
sudo apt-get install ruby
sudo apt-get install ruby-dev

```

b. 安装 jekyll

```
gem install jekyll bundler
gem update jekyll

```

###启动 jekyll

```
jekyll new test
cd test
bundle exec jekyll serve --host 0.0.0.0

```

**使用 screen 后台运行 jekyll**

```
sudo apt-get update
sudo apt-get install screen

screen -S jekyll-serve
bundle exec jekyll serve --host 0.0.0.0

# Ctrl + a + d
```
