---
title: 在Debian 12上安装Dokuwki（使用Apache服务）
description: 
published: 1
date: 2024-08-08T09:21:34.984Z
tags: 服务器, debian 12
editor: markdown
dateCreated: 2024-08-08T09:21:34.984Z
---

> 推广云服务器
> 雨云活动优惠持续发布。
> 现在加入雨云最高享受7折优惠。
> **云服务器特惠:**
> 1h1g 50m 15元/月
> 2h2g 100m 22元/月
> (免备案,预装宝塔,支持windows)
> **虚拟主机特惠:**
> 1h512m 20m 10元/月
> 2h1g 50m 20元/月
> (免备案,预装宝塔)
> 
> ----
> 
> 使用优惠通道 [点此进入](https://www.rainyun.com/ODc4MTY=_?s=Wiki)
> 注册立即领取无门槛5元代金券。
> 您的专属优惠码：`ODc4MTY=`
{.is-success}

# 本文要点

  * 使用cnKiTTY作为ssh客户端
  * 使用WinSCP作为ftp客户端
  * 更新Debian 12内的软件包
  * 安装Apache和必要的运行库
    * 安装和启用模块
  * 重启Apache
  * 下载和配置dokuwiki
  * 配置Apache
    * 启用Apache覆写
    * 配置ssl
  * 其它小问题
    * 清除修订历史
    * 修改上传文件大小限制
    * 解决dokuwiki中文文件名乱码问题
    * 比较简洁的url
    * 更改时区

# 使用cnKiTTY作为ssh客户端

> 本文是以Windows控制云服务器来进行安装。如果不用本文提到的ftp客户端来操作，则方法有所不同。请自行搜索相关命令。
{.is-info}

这是一个KiTTY中文修改版本。与KiTTY相同，是一个免费的Windows Telnet/SSH客户端。更多详细信息，请访问[KiTTY](https://github.com/cyd01/KiTTY)网站及说明。
获取[cnKiTTY](https://github.com/dZ8Lx9OwX/cnKiTTY/releases)

# 使用WinSCP作为ftp客户端

WinSCP 是一款适用于 Windows 的流行免费 SFTP 和 FTP 客户端，是一款功能强大的文件管理器，可提高您的工作效率。它还支持 Amazon S3、FTPS、SCP 和 WebDAV 协议。高级用户可以使用 .NET 程序集自动执行 WinSCP。

[下载WinSCP](https://winscp.net/eng/docs/lang:chs#下载)

注意下载说明，要另外下载中文语言包。

> 本文不再说明如何使用cnKiTTY和WinSCP。
{.is-info}

# 更新Debian 12内的软件包

```bash
sudo apt update
sudo apt upgrade
```

> 安装任何软件包之前先更新系统的软件包有助于减少各种问题的出现。
{.is-success}

> （可选）更改Debian 12的语言为中文：
> 
> ```bash
> sudo apt-get install locales
> sudo dpkg-reconfigure locales
> ```
> 
> 输入完最后一个命令后会有一个区域选择页面，使用<kbd>↑</kbd>或<kbd>↓</kbd>方向键来选择区域。
> 向下找到`zh_CN.UTF-8`选项，空格键选定（出现`*`就是选定了）<kbd>Tab</kbd>键跳出列表，按OK。
> 在下一个屏幕，用空格选定`zh_CN.UTF-8`，<kbd>Tab</kbd>键跳出列表，按OK。
> 要生效必须重启。

如果条件允许，建议重启一下。

```bash
reboot
```

# 安装Apache和必要的运行库

## 安装 Apache

```bash
sudo apt-get install apache2
```

## 安装必要的运行库

```bash
sudo apt install php libapache2-mod-php php-xml php-json php-mbstring php-zip php-intl php-gd
```

## 安装 php-fpm

```bash
sudo apt install php php-fpm
```

## 在 Apache 中启用 proxy_fcgi setenvif 和 PHP 8.2 FPM

```bash
a2enmod proxy_fcgi setenvif
a2enconf php8.2-fpm
```

> 在安装和启用模块后都要[重启Apache](#重启Apache)！
> php后续可能会更新，版本号会更改，请视情况更改命令内的版本号。
{.is-warning}

# 重启Apache

```bash
sudo systemctl restart apache2.service
```

# 下载和配置DokuWiki

## 下载DokuWiki

进入[DokuWiki官网](https://www.dokuwiki.org)下载压缩包文件。
Languages可以只选`zh - 中文`。建议勾选`Upgrade Plugin`方便后续升级DokuWiki。
滑到底部按`Start Download`开始下载压缩包。

## 配置DokuWiki

使用[WinSCP](#使用WinSCP作为ftp客户端)将压缩包上传至''/root''目录。
用命令解压：

```bash
tar xzvf <压缩包文件名>
```

将得到的文件夹（默认是`dokuwiki`，这里以默认文件夹名为例）移动至`/var/www/html`
设置权限：

```bash
sudo chown -R www-data:www-data /var/www/html/dokuwiki
```

# 配置Apache

## 启用Apache覆写

转到以下目录：

```autoconf
/etc/apache2/apache2.conf
```

找到文件里下面的内容：

```autoconf
<Directory /var/www/>

Options Indexes FollowSymLinks
AllowOverride All
Require all granted

</Directory>
```

将`AllowOverride All`改为`AllowOverride None`

启用rewrite模块：

```bash
a2enmod rewrite
```

[重启Apache](#重启Apache)

访问`<你的域名或IP>/install.php`来开始安装。

# 配置ssl（https访问）

安装完就可以使用了，默认是http不加密链接。要想加密传输，请进行下面的步骤。
启用ssl模块：

```bash
a2enmod ssl
```

转到以下目录：

```autoconf
/etc/apache2/sites-available
```

修改`default-ssl.conf`文件：

```autoconf
DocumentRoot /var/www/html/dokuwiki     #你的网站目录

SSLCertificateFile      /xxx/xxx/xxx.cer       #证书目录，后缀名可以是.crt等
SSLCertificateKeyFile   /xxx/xxx/xxx.key    #证书密钥目录，后缀名可以是.pem等
```

启用`default-ssl.conf`文件：

```bash
a2ensite default-ssl
```

[重启Apache](#重启Apache)

> （可选）http自动跳转https：
> 先打开dokuwiki站点目录，找到''.htaccess''文件。
> 将下面的语句前的#号删除：
> 
> ```autoconf
> RewriteCond %{HTTPS} !=on
> ```
> 
> [重启Apache](#重启Apache)
> 由于您浏览器缓存问题，访问http时不会自动跳转到https。可以使用无痕浏览来模拟新访客访问http来测试效果。
{.is-info}

# 其它小问题

这些不会影响正常使用，按需修改。

## 清除修订历史

找到下面的目录：

```autoconf
<dokuWiki目录>/data/attic
<dokuWiki目录>/data/meta
```

清空<mark>里面的内容</mark>。

> 如果启用了discussion插件，评论可能会一并删除。
{.is-warning}

## 修改上传文件大小限制

转到目录：

```autoconf
/etc/php/使用的版本号/fpm
```

修改`php.ini`里的

```ini
post_max_size = 8M    #修改数字
upload_max_filesize = 2M      #修改数字
```

[重启Apache](#重启Apache)

## 解决dokuwiki中文文件名乱码问题

转到目录：

```autoconf
<dokuwiki站点目录>/lib/plugins/config/settings
```

打开`config.metadata.php`
找到

```php
$meta['_basic'] = ['fieldset'];
```

在它的下面加上

```php
$meta['fnencode' ] = array('string');
```

进入你的DokuWiki站点=>管理=>配置设置
找到

`fnencode`

`非 ASCII 文件名的编码方法。`

改为`utf-8`。

## 比较简洁的url

默认的链接形式是`doku.php`后面加上什么东西，这个可以让网址简洁一些。
先打开dokuwiki站点目录，找到`.htaccess`文件。
将下面的语句前的#号删除：

```autoconf
RewriteEngine on

RewriteRule ^_media/(.*)              lib/exe/fetch.php?media=$1  [QSA,L]
RewriteRule ^_detail/(.*)             lib/exe/detail.php?media=$1  [QSA,L]
RewriteRule ^_export/([^/]+)/(.*)     doku.php?do=export_$1&id=$2  [QSA,L]
RewriteRule ^$                        doku.php  [L]
RewriteCond %{REQUEST_FILENAME}       !-f
RewriteCond %{REQUEST_FILENAME}       !-d
RewriteRule (.*)                      doku.php?id=$1  [QSA,L]
RewriteRule ^index.php$               doku.php
```

进入你的DokuWiki站点=>管理=>配置设置
找到高级配置
将

`userewrite`

`使用更整洁的 URL`

改为`.htaccess`

## 更改时区

针对时间日期显示不正确的问题。
转到目录：

```autoconf
/etc/php/使用的版本号/fpm
```

修改`php.ini`里的对应语句为：

```ini
date.timezone = "PRC"    #记得将; 删除
```

[重启Apache](#重启Apache)