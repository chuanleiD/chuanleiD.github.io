---
layout: post
title: "GPT-SoVITS部署与原理"
date:   2024-8-04
tags: [TTS, GPT-SoVITS]
comments: true
author: daichuan
---

本篇文档记录了GPT-SoVITS部署与使用过程，以及其模型架构原理。

<!-- more -->

[TOC]

## 一、GPT-SoVITS部署

[RVC-Boss/GPT-SoVITS: 1 min voice data can also be used to train a good TTS model! (few shot voice cloning) (github.com)](https://github.com/RVC-Boss/GPT-SoVITS/tree/main)

训练环境：`Windows11`，`GPU：4060-8G`，`Cuda：12.1`，`torch：2.3.1+cu121`

### 1.1、选择本地的Python环境

查询本地的Python环境：

```cmd
D:\AI_project\GPT-SoVITS>python --version
Python 3.10.9

D:\AI_project\GPT-SoVITS>py -0p
Installed Pythons found by py Launcher for Windows
 -3.9-64        C:\Users\14766\AppData\Local\Programs\Python\Python39\python.exe *
 -3.8-64        C:\Users\14766\AppData\Local\Programs\Python\Python38\python.exe
 -3.10-64       D:\0python_work\Anaconda\python.exe
```

直接指定运行python脚本的python版本：

```cmd
py -3.8 script.py    # 运行Python 3.8
py -3.10 script.py   # 运行Python 3.10
```

指定创建虚拟环境的python版本：

```cmd
py -3.8 -m venv env38
py -3.10 -m venv env310
```

### 1.2、部署本地虚拟环境

```cmd
python -m venv venv
venv\Scripts\activate

pip install -r requirements.txt --proxy=127.0.0.1:10090
```

<u>pyopenjtalk安装失败问题：</u>

（1）安装`Microsoft Visual Studio`最好<u>使用默认安装路径</u> ！！！

选择使用C++的桌面开发的：

- `MSVC v143- VS2022 C++ x64/x86...`
- ``适用于最新 v143生成工具的 C++ ATL...`
- `Windows 11 SDK(10.0.22000.0)`

配置环境变量：

```
C:\Program Files\Microsoft Visual Studio\2022\Professional\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin
C:\Program Files\Microsoft Visual Studio\2022\Professional\VC\Tools\MSVC\14.40.33807\bin\Hostx64\x64
```

（2）或：直接从`GPT-SoVITS-beta0706`拷贝：`pyopenjtalk`，`pyopenjtalk-0.3.3.dist-info`。

（3）使用现成的wheel：`pip install --only-binary=:all: pyopenjtalk`

### 1.3、无法使用GPU

执行官方文档中的 `pip install -r requirements.txt` ，得到的结果中：

```cmd
torch                     2.4.0
torch-complex             0.4.4
torchaudio                2.4.0
```

对比官方的封装版本中的包环境：

```cmd
torch                         2.0.1+cu118
torchaudio                    2.0.2+cu118
torchvision                   0.15.2+cu118
```

发现，是由于没有使用CUDA版本的pytorch，因此需要重装pytorch：

可以参考：[pytorch配置过程 – Waibibabu's blog (chuanleid.github.io)](https://chuanleid.github.io/记录一次pytorch配置过程/)

```cmd
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121 --proxy=127.0.0.1:8080
```

```cmd
// 运行结果
CUDA 可用。当前使用的设备索引：0
可用的 CUDA 设备数量：1
当前设备名称：NVIDIA GeForce RTX 4060 Laptop GPU
设备 0: NVIDIA GeForce RTX 4060 Laptop GPU
```

------------------

## 二、GPT-SoVITS运行



## 三、GPT-SoVITS架构
