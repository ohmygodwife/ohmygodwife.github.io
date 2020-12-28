---
layout: post
title: "windows command"
date: 2018-08-30 19:38:39 +0800
description: ""
category: command
tags: []
---

#### HTTP下载文件

```
C:\\Windows\\System32\\certutil.exe -urlcache -split -f http://xxx.cn/test.txt

$client = new-object System.Net.WebClient
$client.DownloadFile('https://com/in.txt','D:\out.txt')
```

#### 进程查看与终止

```
wmic path win32_process get ProcessId,commandline > commandline.txt #不指定列名时打印所有字段
taskkill /PID 1234
```

#### 添加程序到右键发送到

右键->创建快捷方式，资源管理器输入shell:sendto，将快捷方式剪切过去

#### Win10状态栏

- [固定状态栏路径](http://www.winwin7.com/JC/11990.html)：`%userprofile%\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\User Pinned\TaskBar`
- [创建快捷启动栏](http://www.zhuangjiba.com/bios/10345.html)：右键状态栏->工具栏->新建工具栏，填入：`%userprofile%\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch`

#### [Admin访问网络映射驱动器](https://blog.csdn.net/jtujtujtu/article/details/51161178)

1. Open the registry editor (regedit.exe)
2. Go to HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System
3. Create a new parameter (DWORD type) with the name `EnableLinkedConnections` and the value `1`, then reboot

#### [Admin访问vmware shared folder](https://superuser.com/questions/1072269/vmware-shared-folder-inaccessible-with-administrator-account)

```
net use Z: "\\vmware-host\Shared Folders"
```

#### [Powershell在此系统上禁止运行脚本](https://www.jianshu.com/p/e5214b3a7627)

```powershell
set-ExecutionPolicy ALLSIGNED
```

#### notepad++

```powershell
regsvr32 NppShell_06.dll #添加到edit with notepad++到右键菜单
regsvr32 /u NppShell_06.dll #移除设置
cmd /k cd "$(CURRENT_DIRECTORY)" & python "$(FULL_CURRENT_PATH)" & EXIT #NppExec
```

#### Firefox

[关闭GET请求到detectportal.firefox.com网站的方法](http://www.mamicode.com/info-detail-1857740.html)：在地址栏中输入`about:config`，搜索`network.captive-portal-service.enabled`，双击改为false即可。

#### [vmware清理虚拟硬盘文件](http://blog.chinaunix.net/uid-13875633-id-4599466.html)

```
vmware-vdiskmanager.exe -k "F:\Fc\FC.vmdk"
```

#### [vmware显示虚拟机繁忙状态，无法关闭](https://blog.csdn.net/qq_44500635/article/details/106789982)

编辑->首选项->设备->启用虚拟打印机

#### VirturalBox更新后休眠的虚拟机报错误E_FAIL (0x80004005) 

右键->清除保存的状态，设置->USB设备->停用USB控制器

#### 卸载默认文件共享

```
net share C$ /delete
```

#### [设置jar文件默认打开方式](https://www.jianshu.com/p/44ab80e80b83)

注册表编辑器，`计算机\HKEY_CLASSES_ROOT\Applications\javaw.exe\shell\open\command`，将`"D:\javahjbl\jdk1.8.0_191\bin\javaw.exe" "%1"`修改为 `"D:\javahjbl\jdk1.8.0_191\bin\javaw.exe" -jar "%1"`，然后把[jar文件默认打开方式设置为javaw.exe](https://blog.csdn.net/lidashent/article/details/109390429)。

- java.exe　　用于启动window console 控制台程序
- javaw.exe　　用于启动 GUI程序
- javaws.exe　　用于web程序
- jvm.dll　　就是java虚拟机规范在windows平台上的一种实现