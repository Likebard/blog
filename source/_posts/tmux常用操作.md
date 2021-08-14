---
title: tmux常用操作
date: 2018-08-14 08:24:40
categories: linux
tags: ["linux", "terminal"]
---

## tmux是什么
tmux（terminal multiplexer）是Linux上的终端复用神器，可从一个屏幕上管理多个终端（准确说是伪终端）。使用该工具，用户可以连接或断开会话，而保持终端在后台运行。

## 我为什么需要它
ssh远程登陆服务器操作时，会出现一段时间不操作，连接断开的情况。对于这种，我喜欢用tmux开启会话，保证连接一直不会断开。而且当你退出服务器后，下次再连接服务器进入tmux时，上次在tmux中对终端的操作状态依然保留着。

## tmux基本结构
tmux的结构包括会话(session)、窗口(window)、窗格(pane)三部分，会话实质是伪终端的集合，每个窗格表示一个伪终端，多个窗格展现在一个屏幕上，这一屏幕就叫窗口。基本结构及状态信息如下图所示：
![tmux介绍](intro.png)

# 如何安装
```
# Ubuntu 或 Debian
$ sudo apt-get install tmux

# CentOS 或 Fedora
$ sudo yum install tmux

# Mac
$ brew install tmux
```

# tmux管理常用快捷键
启动 tmux
退出 exit
新建一个会话 tmux new -s <session-name>
查看所有会话 tmux ls
接入指定会话 tmux attach -t <session-id>/<session-name> 
杀死指定会话 tmux kill-session -t <session-id>/<session-name>
切换会话  tmux switch -t <session-id>/<session-name>
重命名会话 tmux rename-session -t <session-id> <new-name>
关闭所有会话 tmux kill-server
前缀键(prefix) ctrl+b

# 窗口管理常用快捷键
创建一个新窗口 prefix c
重命名当前窗口 prefix ,
列出所有窗口，可进行切换 prefix w
根据显示的内容搜索窗格 prefix f
关闭当前窗口 prefix &

# 窗格管理常用快捷键
水平方向创建窗格 prefix %
垂直方向创建窗格 prefix "
根据箭头方向切换当前工作窗格 prefix Up|Down|Left|Right
根据箭头方向调整当前工作窗格大小 prefix(按住不松手) Up|Down|Left|Right
关闭当前窗格 prefix x