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

#### GCC编译C/C++代码

```sh
$g++ main.cpp -o solve.exe
$gcc main.c -o solve.exe
```

#### [下载Java OpenSDK源代码](https://stackoverflow.com/questions/410756/is-it-possible-to-browse-the-source-of-openjdk-online/410780#410780)

http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/
选类似Mergejdk8u192-b03的版本->browse（左侧导航栏）->src->share->classes->zip（左侧导航栏）

#### [LuaRocks](https://github.com/luarocks/luarocks)管理[Lua](https://www.lua.org/)模块（类似pip）

根据[安装文档](https://github.com/luarocks/luarocks/wiki/Installation-instructions-for-Windows)，下载[最新的win32.zip](http://luarocks.github.io/luarocks/releases)。

```sh
SET PREFIX=D:\Program Files\luarocks
INSTALL /P %PREFIX% /SELFCONTAINED /NOADMIN /L #/L表示安装附带的Lua5.1，如果本地有Lua可以不用
```

将下列三个变量设置到环境变量

```powershell
PATH     :   D:\Program Files\luarocks #用于启动luarocks
LUA_PATH :   D:\Program Files\luarocks\systree\share\lua\5.1\?.lua;D:\Program Files\luarocks\systree\share\lua\5.1\?\init.lua #用于搜索lua文件
LUA_CPATH:   D:\Program Files\luarocks\systree\lib\lua\5.1\?.dll #用于搜索C动态库
```

查找模块地址：https://luarocks.org/。下载安装模块如果需要编译代码则需要先运行VS环境配置

```powershell
>"D:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\vcvarsall.bat"
>luarocks install luabitop #安装bit操作库
>lua5.1 solve.lua #不再报错：lua5.1: solve.lua:1: module 'bit' not found
```



