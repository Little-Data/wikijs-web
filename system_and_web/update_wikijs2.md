---
title: 升级Wiki.js2.0
description: 
published: 1
date: 2024-10-19T15:45:38.414Z
tags: wiki.js, 升级
editor: markdown
dateCreated: 2024-10-19T15:45:38.414Z
---

> 前排提示：本文使用SQLite数据库，且数据有备份（如同步到Github）
{.is-warning}

# 备份配置文件
进入网站的根目录，备份`config.yml`和`db.sqlite`

# 下载最新版本
找一个易于访问的目录，下载最新版本
```bash
wget https://github.com/Requarks/wiki/releases/latest/download/wiki-js.tar.gz
```
# 部署最新版本
删除旧版本的网站目录里的文件（比如网站根目录在wiki文件夹下）
```bash
rm -rf wiki/*
```
解压最新版本到网站目录下
```bash
tar xzf wiki-js.tar.gz -C ./wiki
```
进入网站目录
```bash
cd wiki
```
# 恢复配置文件
将之前备份的`config.yml`和`db.sqlite`放回网站目录下

# 启动服务
启动之前可以先测试一下是否正常远行
```bash
node server
```

> 如果使用了`sqlite`数据库，则可能会有报错，如遇这种情况，可以尝试以下命令来初始化`sqlite3`：
> `npm rebuild sqlite3`
{.is-info}

如正常运行，即可使用进程守护类的软件来托管运行
```bash
systemctl restart wiki
```