---
layout: post
title: "windows下自启动问题，从PE系统下实现自启动"
date:   2024-7-08
tags: [windows系统, 自启动, PE系统]
comments: true
author: daichuan
---

本篇文档讲述在windows环境下，配置HKLM实现程序自启动时遇到的文件读取权限问题及解决方法。以及如何在PE系统下实现本机程序自启动的方法。

<!-- more -->

#### 简单的自启动配置有两种方法：

方法一：添加注册表（需管理员权限）

```
//用户级
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
//管理员权限
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
```

方法二：向启动目录添加快捷方式

```
//自启动目录
C:\Users\Username\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup //目录对特定用户生效， 
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp //对所有用户生效。
```

#### 遇到的问题：

**程序位于`"C:\\test"`下，点击运行、采用方法二运行均正常；采用方法一运行，程序会闪退。**

程序首段如下，采用相对路径创建并写入日志文件，创建的日志文件权限：`0644`。

```go
logFile := "logfile.log"
file, err := os.OpenFile(logFile, os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		log.Println(err)
		time.Sleep(10 * time.Second)
		return
	}
	defer file.Close()
```

<span style="color:#ff4500;">问题在于：HKLM下注册表自启动的程序的工作目录与另外两方式启动的不同。</span>

1、创建一个test测试程序，并配置自启动，验证自启动的程序的工作目录：

```go
func main() {
	currentDir, err := os.Getwd() // 获取当前工作目录
	if err != nil {
		fmt.Println("获取当前工作目录时出错:", err)
		return
	}
	fmt.Println("当前工作目录:", currentDir)
	time.Sleep(50 * time.Second)
} 
```

配置自启动后运行：

方法一的结果：`当前工作目录: C:\Windows\system32`，因此找不到源文件夹下的配置文件，也没有权限创建新的文件。

方法二的结果：`当前工作目录：C:\test\client`

<span style="color:#ff4500;">另一个问题，两种方式运行的程序的身份权限是否会导致该问题？（不会）</span>

```
[GPT4o]：
注册表自启动：程序可能以系统权限或服务权限运行，具体取决于添加注册表项的位置和设置。例如，如果在 HKEY_LOCAL_MACHINE 下添加自启动项，程序会以系统权限运行。如果在 HKEY_CURRENT_USER 下添加自启动项，程序会以当前用户的权限运行。
系统权限运行的程序通常可以访问和写入用户创建的文件，但可能会受到某些访问控制列表 (ACL) 和权限设置的限制。权限设置为 0644 的文件意味着所有者可以读写，其他用户只能读取。因此，如果程序以系统权限运行，通常能够写入这些文件，前提是系统账户有足够的权限。
```

2、创建一个test2测试程序，并配置自启动，验证自启动身份不同导致的0644权限问题：

```go
func main() {
	logFile := "C:\\test\\logfile.log"
	file, err := os.OpenFile(logFile, os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0644)
	if err != nil {
		log.Println(err)
		time.Sleep(10 * time.Second)
		return
	}
	defer file.Close()
	fmt.Println("logfile Opened")
	time.Sleep(50 * time.Second)
}
```

首先，通过用户直接点击的方式运行程序，然后配置自启动后重启。

方法一的结果：`logfile Opened`

方法二的结果：`logfile Opened`

## 一、PE系统下实现程序自启动方法

### 0、PE系统

> [!TIP]
>
> <span style="color:#1f883d;">由于只能接触到PE系统，~~因此，可能无法提供实现【主系统下程序自启动的一键运行的程序】~~，可以以`Firpe`系统为例，提供解决方案。</span>

采用`Firpe`工具：https://firpe.cn/page-247

（0）首先下载[FirPE](http://firpe.cn/page-247)，然后插入U盘。

（1）点击ISO制作。

（2）点击全新制作。

（3）将制作的`FirPE-V1.9.1.iso`，`FirPE-V1.9.1-数据区文件.zip`拷贝到U盘中。

（4）重启后，进入PE系统。

<img src="C:\mysoft\typora\typora_picture\@SGAS0YCJZR2XEWZ75PMR.png" alt="http://firpe.cn/wp-content/uploads/2018/06/@SGAS0YCJZR2XEWZ75PMR.png" style="zoom:80%;" /> 

----------

### 1、将程序放在指定文件夹下：

```
//自启动目录
C:\Users\Username\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup //目录对特定用户生效， 
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp //对所有用户生效。
```

自动化程序：在PE下点击运行`_pe_autoRun2.exe`，需要PE系统存在`cmd`。

配置文件`_pe_autoRun.json`：

```json
{
    "name": "test", # 注册表中字符串的名称
    "path": "C:\\Program Files (x86)\\Windows Media", # 隧道程序client希望放在目标系统中的文件夹位置
    "fileName": "testRun.exe", # 隧道程序client本身的名称
    "config": "config.enc" # 隧道程序client的配置文件名称
}
```

---------

### 2、将程序添加到自启动注册表

```
//程序位置：C:\Program Files (x86)\Microsoft\testRun.exe
//注册表位置：
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

手动操作流程参考：[PE下修改注册表 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/32487069)

> （1）首先 win+R 打开注册表，打开`HKEY_LOCAL_MACHINE`。
>
> （2）然后加载配置单元：`C:\Windows\System32\config\SOFTWARE`（命名为local_software）。
>
> （3）然后打开：`\Microsoft\Windows\CurrentVersion\Run`，添加字符串。
>
> （4）最后，卸载配置单元，上传配置。

自动化程序执行方法：

配置文件`_pe_autoRun.json`：

```json
{
    "name": "test", # 注册表中字符串的名称
    "path": "C:\\Program Files (x86)\\Windows Media", # 隧道程序client希望放在目标系统中的文件夹位置
    "fileName": "testRun.exe", # 隧道程序client本身的名称
    "config": "config.enc" # 隧道程序client的配置文件名称
}
```

点击运行`_pe_autoRun2.exe`：代码逻辑：读取json文件中的参数，调用`_pe_autoRun_reg.bat`实现注册表写入。

> 【PS】：
