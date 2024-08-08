---
title: Win11恢复经典右键菜单
description: 新版不太好用
published: 1
date: 2024-08-08T08:48:23.138Z
tags: windows, 注册表, 系统调整
editor: markdown
dateCreated: 2024-08-08T08:48:23.138Z
---

> 此操作会重启资源管理器，注意保存工作！
{.is-warning}

按Win+r打开运行，输入：
```batch
reg add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve

taskkill /f /im explorer.exe & start explorer.exe
```

恢复新右键菜单：

```batch
reg delete "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}" /f

taskkill /f /im explorer.exe & start explorer.exe
```