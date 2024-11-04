---
title: Win去掉创建快捷方式时的「-快捷方式」后缀
description: 
published: 1
date: 2024-11-04T08:38:02.470Z
tags: windows, 注册表, 系统调整
editor: markdown
dateCreated: 2024-08-02T14:41:40.467Z
---

按<kbd>Win</kbd>+<kbd>r</kbd>打开运行，输入`regedit`
定位到以下路径：

```batch
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer
```

找到右侧名为`link`的二进制值
修改为`00 00 00 00`

> 一定是4组0否则无效！
{.is-warning}

恢复时只需要前两位不全为`00`即可。

# CMD 命令

CMD（管理员身份）执行以下命令也可以一键完成。

```batch
reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer" /v "link" /d 00000000 /t REG_binary /f
```

> 不管使用哪种方法，一定要重启系统或`explorer`，否则重新操作！
{.is-warning}