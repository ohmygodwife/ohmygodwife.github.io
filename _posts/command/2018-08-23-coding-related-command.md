---
layout: post
title: "coding related command"
date: 2018-08-23 15:40:13 +0800
description: ""
category: command
tags: []
---

## C/C++

#### [Visual Studio编译C/C++代码](https://msdn.microsoft.com/zh-cn/library/f2ccy3wt.aspx)

安装Visual Studio Community时`工作负载栏`选择`C++桌面开发`，或者`单个组件`栏选择`MSVC x64/x86生成工具`和`Windows 10 SDK`。

```powershell
>"C:\Users\tyler\AppData\Local\Programs\Common\Microsoft\Visual C++ for Python\9.0\vcvarsall.bat" amd64 #不指定默认32bit
>D:\ctf\software\VisualStudio\2019\Community\VC\Auxiliary\Build\vcvarsall.bat x86
>cl.exe /Fe:solve.exe main.cpp #/Fe:指定输出文件名
```

const wchar_t*类型实参与PTSTR类型形参不兼容：右键项目->属性->C/C++->语言->符合模式选否

#### GCC编译C/C++代码

```sh
$g++ main.cpp -o solve.exe
$gcc main.c -o solve.exe
```

#### OpenSSL编程

```shell
#C/C++ --> code generation
running library /MT #static link
#Linker --> Additional Dependencies
libssl32MT.lib;libcrypto32MT.lib;Crypt32.lib
#Generate ssl certificate(crt)
openssl genrsa -des3 -out server.key 1024 #private key file
openssl req -new -x509 -key server.key -out ca.crt -days 3650 #CA public key
openssl req -new -key server.key -out server.csr #Certificate Signing Request
#generate server.crt:
openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
#OR
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey server.key -CAcreateserial -out server1.crt
cat server.key server.crt > server.pem
openssl dhparam -out dh2048.pem 2048
```

#### 查动态依赖库

```
dumpbin /dependents a.exe
```

## Python

#### Pip安装指定源

```shell
pip install -i http://pypi.douban.com/simple --trusted-host pypi.douban.com software
--default-timeout=100 #报socket.timeout错误时加上
pip config set global.index-url https://pypi.douban.com/simple #设置全局
```

#### Python通过setup.py安装卸载

```sh
$python setup.py install --record files.txt #记录安装后文件的路径
$cat files.txt | xargs rm -rf  #删除这些文件
```

#### [Python Import自定义模块](https://blog.csdn.net/v_xchen_v/article/details/80393967)

如果模块所在目录存在一个`__init__.py`文件，则该目录会被python认为是一个包，否则就是普通的一个目录。

- 模块中直接修改`sys.path`的值，只对当前进程有效，避免冲突

```python
lib_dir = '/mnt/hgfs/snippet/ctf/pwn' #to import personal py, for instance file.py
if lib_dir not in sys.path:
  sys.path.append(lib_dir)
from FILE import *
```

- 添加`pth`文件

```shell
#将自定义模块所在的目录写到/usr/local/lib/python2.7/dist-packages下某个pth文件
/usr/local/lib/python2.7/dist-packages$ sudo echo "/mnt/shared/py/linux" > pwn-file.pth
```

- 添加环境变量`PYTHONPATH`，不同路径以逗号分隔，所定义的路径会自动加到`sys.path`中

#### scrapy

[spider文件中不能有comment注释](https://blog.csdn.net/qq_36893628/article/details/105722472)，包括`#`和`'''`，否则会抛`IndentationError: unexpected indent`错误！

## Java

#### [下载Java OpenSDK源代码](https://stackoverflow.com/questions/410756/is-it-possible-to-browse-the-source-of-openjdk-online/410780#410780)

http://hg.openjdk.java.net/jdk8u/jdk8u/jdk/
选类似Mergejdk8u192-b03的版本->browse（左侧导航栏）->src->share->classes->zip（左侧导航栏）

#### [Maven生成可直接运行jar包](https://blog.csdn.net/xiao__gui/article/details/47341385)

```xml
<build>
  <plugins>
    <plugin>
      <artifactId>maven-assembly-plugin</artifactId>
      <configuration>
        <archive>
          <manifest>
            <mainClass>plc.tz.tzcloud.App</mainClass>
          </manifest>
        </archive>
        <descriptorRefs>
          <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
      </configuration>
      <executions>  
	    <execution>  
	      <id>make-assembly</id>  
	      <phase>package</phase>  
	      <goals>  
	        <goal>single</goal> <!--if not, need to run: mvn package assembly:single-->
          </goals>  
	    </execution>  
	  </executions>
    </plugin>
  </plugins>
</build>
```

#### [UnsupportedEncodingException](https://javaee.github.io/javamail/FAQ#imapserverbug)

利用[Service Provider Interface扩展加载机制](https://blog.csdn.net/qq893555741/article/details/84911412)，自定义

```java
==> java.nio.charset.Charset.java <==
private static Charset lookup2(String charsetName) {
  if ((cs = standardProvider.charsetForName(charsetName)) != null ||
            (cs = lookupExtendedCharset(charsetName))           != null ||
            (cs = lookupViaProviders(charsetName))              != null)
            //依次从标准、扩展、自定义的Provider中查找

==> resources/META-INF/services/java.nio.charset.spi.CharsetProvider <==
net.freeutils.charset.CharsetProvider //先使用jcharset
plc.tz.tzcloud.util.UnknownCharsetProvider

==> UnknownCharsetProvider.java <==
package plc.tz.tzcloud.util;
public class UnknownCharsetProvider extends CharsetProvider {
}
```

## Lua

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

## MySQL

#### Ubuntu安装MySQL

```shell
sudo apt-get install mysql-server
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
# bind-address = 127.0.0.1 #注释这一行
service mysql restart
```

## Nodejs

#### [开发electron程序的npm准备工作](http://www.6yang.net/articles_view_1494_26.html)

```
npm config set registry https://registry.npm.taobao.org/
npm config set electron_mirror http://npm.taobao.org/mirrors/electron/
npm config edit #verify
```

