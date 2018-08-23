---
layout: post
title: "coding related command"
date: 2018-08-23 15:40:13 +0800
description: ""
category: command
tags: []
---

#### [Visual Studio编译C/C++代码](https://msdn.microsoft.com/zh-cn/library/f2ccy3wt.aspx)

```powershell
>"D:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\vcvarsall.bat" amd64 #不指定默认32bit
>cl.exe /Fe:solve.exe main.cpp #/Fe:指定输出文件名
```

GCC编译C/C++代码

```sh
$g++ main.cpp -o solve.exe
$gcc main.c -o solve.exe
```

