layout: post
title: "SigThief——为exe程序添加一个无效签名"
date:   2024-7-10
tags: [程序伪装, 数字签名]
comments: true
author: daichuan

本篇文档讲述如何在windows环境下，为windows环境下的exe可执行文件打一个从其他程序上拿到的（无效）签名。

<!-- more -->

**原项目链接**：[secretsquirrel/SigThief: Stealing Signatures and Making One Invalid Signature at a Time (github.com)](https://github.com/secretsquirrel/SigThief)

**行为原因**：多年来，SigThief的作者在与反病毒软件的测试中注意到，无论签名是否有效，还是对 PE 签名的优先级，每种反病毒软件的表现都不尽相同。<u>有些反病毒软件厂商会优先考虑某些证书颁发机构，而不检查签名是否有效；还有些厂商只检查证书表是否填入了某些值</u>。这个工具可以快速完成反病毒软件针对证书检查的测试。简而言之，它能从已签名的 PE 文件中提取一个签名，并将其附加到另一个文件中，修复证书表以签署该文件。当然，这不是一个有效的签名！

**部署方式**：

clone一下项目即可。

```python
# 环境依赖:
import sys
import struct
import shutil
import io
from optparse import OptionParser
```

**运行方法：**

```
venv\Scripts\activate.bat
python sigthief.py -i signedPE.exe -t targetPE.exe -o outputPE.exe
```

```
选项:
  -h、--help 显示帮助信息并退出
  -i FILE, --file=FILE 输入文件（已签名的PE文件，从他身上获取签名）
  -t TARGETFILE, --target=TARGETFILE 要附加签名的文件
  -o OUTPUTFILE, --output=OUTPUTFILE 输出文件    
  
  -r, --rip 从输入文件中撕下签名（并保存在磁盘）
  -a, --add 向目标文件添加签名（使用来自磁盘的二进制签名）
  
  -s SIGFILE, --sig=SIGFILE 来自磁盘的二进制签名
  
  -c，--checksig 文件，检查是否签名；不验证签名有效性
  -T, --truncate 截断签名（即删除一个程序的签名）
```
