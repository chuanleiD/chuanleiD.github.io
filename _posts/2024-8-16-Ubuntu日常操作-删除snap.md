---
layout: post
title: "Ubuntu日常操作-删除snap"
date:   2024-8-16
tags: [Ubuntu, Linux, snap]
comments: true
author: chenliang
---

本篇文档记录了 Ubuntu日常操作-删除snap 的细节。

参考：[Ubuntu安装教程 - 我的博客 (hsj01wzonline.top)](https://blog.hsj01wzonline.top/posts/myblog38/)，https://www.cnblogs.com/learner-and-helper-YZY/p/17654961.html

<!-- more -->

[TOC]

## 一、删除 snap

展示已安装的snap包：

```bash
snap list
```

删掉所有的已经安装的 Snap 软件：

```bash
for p in $(snap list | awk '{print $1}'); do
  sudo snap remove $p
done
```

（一般需要执行两次，提示如下则正确执行：`No snaps are installed yet. Try 'snap install hello-world`）

```bash
sudo systemctl stop snapd
sudo systemctl disable --now snapd.socket
sudo apt autoremove --purge snapd

rm -rf ~/snap
sudo rm -rf /snap
sudo rm -rf /var/snap
sudo rm -rf /var/lib/snapd
sudo rm -rf /var/cache/snapd
```

## 二、禁止自动安装 Snap 服务

```bash
sudo vim /etc/apt/preferences.d/nosnap.pref
```

文件中粘贴一下内容并保存

```
Package: snapd 
Pin: release a=* 
Pin-Priority: -10 
```

然后运行如下命令

```bash
sudo apt update
```

## 三、安装apt版gnome应用商店

```bash
sudo apt install --install-suggests gnome-software
```

<img src="https://cdn.jsdelivr.net/gh/chuanleiD/chuanleiD.github.io//images/202408161749331.png" style="zoom: 40%;" /> 

## 四、APT 版本的 Firefox

[Ubuntu安装教程 - 我的博客 (hsj01wzonline.top)](https://blog.hsj01wzonline.top/posts/myblog38/)
