---
title: 显示Windows登录过程详细信息
description: 查看登录过程到底做了什么
published: 1
date: 2024-08-02T14:42:39.865Z
tags: windows, 注册表, 系统调整
editor: markdown
dateCreated: 2024-08-02T13:19:31.956Z
---

按<kbd>Win</kbd>+<kbd>r</kbd>打开运行，输入`regedit`
定位到以下路径：
```batch
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
```
创建一个名为`VerboseStatus`的`32 位 DWORD 值`，并将其设为`1`。