---
layout: post
title: "tensorflow学习"
date:   2024-8-16
tags: [web, Api, video]
comments: true
author: chenliang
---

本篇文档用于记录tensorflow学习。

<!-- more -->

[TOC]

## 一、环境部署

[使用 pip 安装 TensorFlow](https://www.tensorflow.org/install/pip?hl=zh-cn)

- 从 TensorFlow 2.1.0 版开始，此软件包需要 `msvcp140_1.dll` 文件。该可再发行软件包随附在 Visual Studio 2019 中，但可以单独安装： [Microsoft Visual C++ 下载](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads)页面。

- 确保在 Windows 上[启用了长路径](https://superuser.com/questions/1119883/windows-10-enable-ntfs-long-paths-policy-option-missing)。

```cmd
PS C:\Users\14766> reg query "HKLM\SYSTEM\CurrentControlSet\Control\FileSystem" /v LongPathsEnabled

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem
    LongPathsEnabled    REG_DWORD    0x1
```

- 创建虚拟环境并安装

```cmd
conda create --name tensorflow_env python=3.9
pip install --upgrade tensorflow

// 验证方法
python -c "import tensorflow as tf;print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
```

- 配置jupyter

```vmd
conda activate tensorflow_env
conda install ipykernel
 
python -m ipykernel install --user --name=tensorflow_env --display-name="tensorflow2 (py3.9)"
jupyter kernelspec list 
```

-------

## 二、数据处理



## 三、tensorflow
