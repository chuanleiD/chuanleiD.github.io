---
layout: post
title: "安装kata安全容器运行时，解决docker pull，containerd pull代理配置问题"
date:   2024-7-17
tags: [docker, proxy, containerd]
comments: true
author: daichuan
---

本篇文档记录了安装kata安全容器运行时，以及使用代理解决docker pull失败问题的过程。

<!-- more -->

[TOC]

## 一、使用代理解决docker pull失败问题

参考：https://juejin.cn/post/7005407213615480839

（很多国内的镜像服务都关了）

```bash
sudo vim /etc/systemd/system/multi-user.target.wants/docker.service

# 然后在[service] Restart行下面加入代理的配置，比如：
Environment=HTTP_PROXY=http://192.168.xxx.xxx:xxx
Environment=HTTPS_PROXY=http://192.168.xxx.xxx:xxx
Environment=NO_PROXY=localhost,127.0.0.1

# 其中，Environment=HTTPS_PROXY（程序本身使用的协议）= http（代理使用的协议）://192.168.xxx.xxx:xxx（代理的ip与port）
```

之后重启即可：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 二、正确的官方文档

另外参考：https://www.cnblogs.com/abc1069/p/17496240.html

可知，设置环境变量`HTTP_PROXY` 和 `HTTPS_PROXY`的办法无效，配置 `~/.docker/config.json` 文件的方法无效。

关于 systemd 配置代理服务器的**官方文档**在这里，原文说：

```
The Docker daemon uses the HTTP_PROXY, HTTPS_PROXY, and NO_PROXY environmental variables in its start-up environment to configure HTTP or HTTPS proxy behavior. You cannot configure these environment variables using the daemon.json file.

This example overrides the default docker.service file.

If you are behind an HTTP or HTTPS proxy server, for example in corporate settings, you need to add this configuration in the Docker systemd service file.
```

这段话的意思是，docker daemon 使用 `HTTP_PROXY`, `HTTPS_PROXY`, 和 `NO_PROXY` 三个环境变量配置代理服务器，但是你需要在 systemd 的文件里配置环境变量，而不能配置在 `daemon.json` 里。



## 三、使用解决containerd pull失败问题

<u>**失败**</u>，配置文件：`/etc/systemd/system/containerd.service.d/http-proxy.conf`

<u>**失败**</u>，配置文件：`/etc/systemd/system/multi-user.target.wants/containerd.service`

解决方案：使用 Docker 下载镜像并让 containerd 使用，您需要遵循以下步骤：

1. 使用 Docker 拉取镜像：

   ```bash
   docker pull busybox:latest
   ```

2. 将 Docker 镜像保存为 tar 文件：

   ```bash
   docker save busybox:latest -o busybox.tar
   ```

3. 使用 ctr（containerd 的命令行工具）导入镜像：

   ```bash
   sudo ctr image import busybox.tar
   ```

4. 验证镜像是否已导入到 containerd：

   ```bash
   sudo ctr image ls
   ```

### 脚本如下：

使用方法：将`docker pull ***`，改为 `containerd_pull ***`

```bash
#!/bin/bash

# 检查是否提供了镜像名称参数
if [ $# -eq 0 ]; then
    echo "请提供镜像名称作为参数"
    echo "用法: $0 <镜像名称>"
    exit 1
fi

# 获取镜像名称
IMAGE_NAME=$1

# 使用 Docker 拉取镜像
echo "正在使用 Docker 拉取镜像: $IMAGE_NAME"
docker pull $IMAGE_NAME

if [ $? -ne 0 ]; then
    echo "拉取镜像失败"
    exit 1
fi

# 获取实际的镜像 ID
ACTUAL_IMAGE_ID=$(docker inspect --format='{{.Id}}' $IMAGE_NAME)

# 获取镜像的所有标签
IMAGE_TAGS=$(docker inspect --format='{{join .RepoTags ", "}}' $ACTUAL_IMAGE_ID)

echo "实际拉取的镜像标签: $IMAGE_TAGS"

# 生成一个唯一的文件名用于 tar 文件
TAR_FILE="$IMAGE_NAME.tar"

# 将镜像保存为 tar 文件
echo "正在将镜像保存为 tar 文件: $TAR_FILE"
# docker save 使用$IMAGE_TAGS，不要使用$ACTUAL_IMAGE_ID
# 当使用纯 ID 保存的镜像被导入时，containerd 可能无法正确解析或识别这个镜像。
docker save $IMAGE_TAGS -o $TAR_FILE 

if [ $? -ne 0 ]; then
    echo "保存镜像失败"
    exit 1
fi

# 将 tar 文件导入到 containerd
echo "正在将镜像导入到 containerd"
sudo ctr image import $TAR_FILE

if [ $? -ne 0 ]; then
    echo "导入到 containerd 失败"
    exit 1
fi

# 删除 tar 文件
echo "正在清理临时文件"
rm $TAR_FILE

# 验证镜像是否已成功导入到 containerd
echo "验证镜像是否已导入到 containerd:"
sudo ctr image ls | grep $ACTUAL_IMAGE_ID
echo "操作完成"
```

## 四、安装kata安全容器运行时

kata一般不与docker一起使用：kata2.x后，kata去掉了docker的cli，不能通过docker启动kata runtime容器，需要docker in Docker技术。[kata-containers/docs/how-to/how-to-run-docker-with-kata.md](https://github.com/kata-containers/kata-containers/blob/main/docs/how-to/how-to-run-docker-with-kata.md)

### 使用containerd+kata部署：

[kata-containers/docs/install/container-manager/containerd/containerd-install.md](https://github.com/kata-containers/kata-containers/blob/main/docs/install/container-manager/containerd/containerd-install.md)

[kata-containers/utils/README.md](https://github.com/kata-containers/kata-containers/blob/main/utils/README.md)

1、请首先安装docker（一整套套件，省事了），并可以使用docker+runc运行容器。

2、下载：kata-static-3.6.0-amd64.tar.xz，[https://github.com/kata-containers/kata-containers/releases/tag/3.6.0](https://github.com/kata-containers/kata-containers/releases/tag/3.6.0)

```bash
# 解压，可以在解压后转移至根目录
tar -xvf kata-static-3.6.0-amd64.tar.xz
```

3、执行安装程序

```bash
sudo chmod 755 opt/kata/bin/kata-manager
sudo opt/kata/bin/kata-manager -o -K kata-static-3.6.0-amd64.tar.xz
```

4、配置containerd

```bash
sudo vim /etc/containerd/config.toml
# 然后写入：
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "kata"
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.kata]
          runtime_type = "io.containerd.kata.v2"
# 重启服务
sudo systemctl daemon-reload
sudo systemctl restart containerd
```

5、测试

```bash
# 先执行 containerd_push 命令获取本地镜像
image="docker.io/library/ubuntu:latest"
sudo ctr image pull "$image"
sudo ctr run --runtime "io.containerd.kata.v2" --rm -t docker.io/library/ubuntu:latest ubuntu-kata /bin/bash
```

## 五、Containerd入门

[containerd容器运行时快速入门使用指南 - 尹正杰 - 博客园 (cnblogs.com)](https://www.cnblogs.com/yinzhengjie/p/18058010)
