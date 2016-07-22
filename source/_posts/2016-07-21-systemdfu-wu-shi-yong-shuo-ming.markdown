---
layout: post
title: "systemd服务使用说明"
date: 2016-07-21 10:48:24 +0800
comments: true
categories: systemd
---
systemd服务使用说明
<!--more-->

# 1.使用systemctl --user enable 命令启动服务
```
systemctl --user enable /your/service/path
```
创建的服务在 ~/.config/

# 2.服务控制
```
systemctl --user start your.service    #启动服务
systemctl --user status your.service   #查看服务状态
systemctl --user stop your.service     #停止服务
systemctl --user restart your.service  #重启服务
```

# 3.删除服务
首先将服务停止，然后删除软链。