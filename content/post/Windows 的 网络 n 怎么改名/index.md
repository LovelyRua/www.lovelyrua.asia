---
title: 'Windows 的 "网络 n" 怎么改名'
description: "修改注册表 ProfileName, 自定义 Windows 网络名称"
date: 2026-10-13 08:03:00+0800
categories:
    - IT-Techs
tags:
    - Windows
    - Registry
    - Networking
---

Windows 有时会把一个网络显示成 `网络 2`, `网络 3`, 甚至更大的编号

强迫症患者看到了难免感到手痒

想把它改成 `网络` 或 `LAN`, 不需要重置网络, 可以修改对应配置档的 `ProfileName` 注册项

## 先认准要改的是哪个网络

打开 PowerShell, 输入:

```powershell
Get-NetConnectionProfile | Select-Object Name, InterfaceAlias, NetworkCategory
```

会输出类似:

```text
Name       InterfaceAlias  NetworkCategory
----       --------------  ---------------
网络 3     以太网          Private
```

这里要改的是 `Name`, `InterfaceAlias` 是网卡名称, `NetworkCategory` 是公用或专用网络类型

记下当前连接显示的名字, 等会儿用它在注册表里找对应的配置档

## 在注册表里改名

按 `Win + R`, 输入 `regedit`, 打开注册表编辑器, 定位到:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles
```

`Profiles` 下是一串以 `{GUID}` 命名的子项

逐个点开, 看右侧的 `ProfileName`, 找到值为刚才那个 `网络 3` 的项

先右键这个 `{GUID}` 子项, 选择 `导出`, 留一份 `.reg` 备份

然后双击右侧的 `ProfileName`, 把数值数据改成想要的名字, 比如 `Cisco LAN`

修改后重新打开网络设置确认

如果仍显示旧名字, 断开再连接一次, 或重启 Windows

## 参考

- [Microsoft: Get-NetConnectionProfile](https://learn.microsoft.com/en-us/powershell/module/netconnection/get-netconnectionprofile?view=windowsserver2025-ps)
- [Microsoft Q&A: How to rename network connections](https://learn.microsoft.com/en-us/answers/questions/2843319/how-to-rename-the-3-network-connections-i-have-on)
