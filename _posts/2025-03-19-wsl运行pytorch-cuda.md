---
layout: post
title: "wsl运行pytorch-cuda"
date:   2025-03-19
tags: [cuda,linux,wsl,pytorch]
comments: true
author: chenliang
---

本篇文档用于记录 `wsl` 运行 `pytorch-cuda` 时遇到的一个问题。

<!-- more -->

确保已经安装nvidia驱动，只需在Windows上安装好GPU驱动程序，wsl2上便可以获得驱动的支持。

在wsl2中输入`nvidia-smi`查看

```bash
(ccic_env) chenliang@daichuan:~$ nvidia-smi
Wed Mar 19 21:21:25 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 555.49                 Driver Version: 555.94         CUDA Version: 12.5     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4060 ...    On  |   00000000:01:00.0  On |                  N/A |
| N/A   57C    P8              8W /   80W |    1686MiB /   8188MiB |     34%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

安装wsl版本的CUDA Toolkit

[CUDA Toolkit 11.6 Downloads | NVIDIA Developer](https://developer.nvidia.com/cuda-11-6-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_local)

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.6.0/local_installers/cuda-repo-wsl-ubuntu-11-6-local_11.6.0-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-11-6-local_11.6.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-wsl-ubuntu-11-6-local/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
```

如果遇到： `libcufile-11-6 : Depends: liburcu6 but it is not installable`

参考：[nvidia - 无法在 Ubuntu 22.04 WSL2 上安装 CUDA - 询问 Ubuntu](https://askubuntu.com/questions/1407962/unable-to-install-cuda-on-ubuntu-22-04-wsl2)

是否安全？难说

```bash
ppa:cloudhan/liburcu6提供适用于 Ubuntu 22.04 的 liburcu6 的正向端口。

sudo add-apt-repository ppa:cloudhan/liburcu6
sudo apt update
sudo apt install liburcu6
```

