---
layout: post
title: "windows command"
date: 2018-08-30 19:38:39 +0800
description: ""
category: command
tags: []
---

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

#### VirturalBox更新后休眠的虚拟机报错误E_FAIL (0x80004005) 

右键->清除保存的状态，设置->USB设备->停用USB控制器