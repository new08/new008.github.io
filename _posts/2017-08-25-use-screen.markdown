---
layout: post
title:  "使用 screen"
date:   2017-08-25 11:22
categories: post
tags: linux screen
---

# 使用 screen

**环境** ubuntu 17

screen 是一个Linux上的终端多路复用器，可以在一个终端内访问多个独立的登录会话。

### 安装 screen

  
``` 
sudo apt-get update
sudo apt-get install screen

```

### screen 常用命令

|命令|说明|
|:--|:--|
|screen -h|帮助文档|
|screen -list|查看当前存在的screen|
|screen -S [sockname]|创建一个 screen|
|screen -r [session]|切换到指定 screen|
|screen -wipe|清理 Dead socket|

下列命令在screen内使用

|命令|说明|
|:--|:--|
|Ctrl + a + d|退出当前 socket|

### 示例

**使用 screen 启动服务程序**
```
screen -S jekyll-serve
bundle exec jekyll serve --host 0.0.0.0

Ctrl + a + d

```

**socket 管理**
```
root@who2:/usr/software/new008.github.io# screen -list
There are screens on:
	25438.pts-0.who2	(08/25/2017 10:50:37 AM)	(Detached)
	25076.jekyll-serve	(08/25/2017 09:58:30 AM)	(Detached)
2 Sockets in /var/run/screen/S-root.
root@who2:/usr/software/new008.github.io# kill -9 25438
root@who2:/usr/software/new008.github.io# ls
404.html  about.md  _config.yml  Gemfile  Gemfile.lock  index.md  _posts  README.md  _site
root@who2:/usr/software/new008.github.io# screen -v
Screen version 4.03.01 (GNU) 28-Jun-15
root@who2:/usr/software/new008.github.io# screen -list
There are screens on:
	25076.jekyll-serve	(08/25/2017 09:58:30 AM)	(Detached)
	25438.pts-0.who2	(08/11/2017 10:10:55 AM)	(Dead ???)
Remove dead screens with 'screen -wipe'.
2 Sockets in /var/run/screen/S-root.
root@who2:/usr/software/new008.github.io# kill -9 25076
root@who2:/usr/software/new008.github.io# screen -list
There are screens on:
	25438.pts-0.who2	(08/11/2017 10:10:55 AM)	(Dead ???)
	25076.jekyll-serve	(08/11/2017 10:10:55 AM)	(Dead ???)
Remove dead screens with 'screen -wipe'.
2 Sockets in /var/run/screen/S-root.
root@who2:/usr/software/new008.github.io# screen -wipe
There are screens on:
	25438.pts-0.who2	(08/11/2017 10:10:55 AM)	(Removed)
	25076.jekyll-serve	(08/11/2017 10:10:55 AM)	(Removed)
2 sockets wiped out.
No Sockets found in /var/run/screen/S-root.

root@who2:/usr/software/new008.github.io# screen -list
No Sockets found in /var/run/screen/S-root.

root@who2:/usr/software/new008.github.io# 


```
