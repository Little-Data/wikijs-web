---
title: 批处理脚本
description: 
published: 1
date: 2024-08-27T08:58:02.639Z
tags: windows, 脚本, 批处理
editor: markdown
dateCreated: 2024-08-27T08:58:02.639Z
---

# 批处理脚本
自行复制源代码，再粘贴到文本文件，然后改`.txt`为`.bat`
大部分脚本是与要处理的文件放在一起。
某些包含`ffmpeg`命令，请到官网下载并添加到系统变量。
- [Download FFmpeg](https://ffmpeg.org/download.html)
{.links-list}

# awb解包的bin改成hca

```batch
ren *.bin *.hca
```

# srt批量转换ass软字幕

```batch
for %%i in ("*.srt") do ffmpeg -i %%i "%%~ni.ass
pause
```

# wav改wem

```batch
ren *.wav *.wem
```

# 给文件加后缀

```batch
ren *. *.ab
```
> 到这里，你应该知道了`ren`的基本用法：`ren <原有部分> <要改为什么>`
{.is-success}

# 批量去视频软字幕

```batch
for %%i in ("*.mkv") do ffmpeg -i %%i -c:v copy -c:a copy -sn "%%~ni.mp4
pause
```
> `*.mkv`在此处的意思是批量处理当前目录下所有后缀名为`.mkv`的文件，如果要处理的文件不是该后缀名，可以修改该部分。
> 例如：批量去`.mp4`软字幕，可以将`"*.mkv"`改为`"*.mp4"`
{.is-info}

# 批量删除后32位字符

```batch
@echo off
::Deep Lee
setlocal enabledelayedexpansion
for %%f in (*.usm) do ( 
echo %%f
set name=%%f
ren !name! !name:~0,-36%!.usm
)
pause
```

# 批量删除前N个字符

```batch
@echo off
setlocal enabledelayedexpansion
 
::批量去掉文件名前N个字符，如果有文件夹会搜索文件夹下的每个文件进行修改
set /p format=请输入需要操作的文件格式：wav
set /p deletenum=请输入需要删除文件名前多少个字符：
for /r %%i in (.) do (
    for /f "delims=" %%a in (' dir /b "%%i\*.%format%" 2^>nul ') do (
		set "t=%%~na"
        ren "%%i\%%a" "!t:~%deletenum%!%%~xa"
    )
)
 
pause
```

# 删除usm视频字符

```batch
@echo off
Setlocal Enabledelayedexpansion
set "str=_40534656"
for /f "delims=" %%i in ('dir /b *.*') do (
set "var=%%i" & ren "%%i" "!var:%str%=!")
```

# 删除usm音频字符

```batch
@echo off
Setlocal Enabledelayedexpansion
set "str=_40534641"
for /f "delims=" %%i in ('dir /b *.*') do (
set "var=%%i" & ren "%%i" "!var:%str%=!")
```

# 删除括号

```batch
@Echo Off&SetLocal ENABLEDELAYEDEXPANSION

FOR %%a in (*) do (

set "name=%%a"

set "name=!name: (=!"

set "name=!name:)=!"

ren "%%a" "!name!"

)

exit
```

# 替换文字

```batch
@echo off
set /p str1= 请输入要替换的文件(文件夹)名字符串（可替换空格）：
set /p str2= 请输入替换后的文件(文件夹)名字符串（若删除直接回车）：
echo.
echo 正在操作中，请稍候……
for /f "delims=" %%a in ('dir /s /b ^|sort /+65535') do (
if "%%~nxa" neq "%~nx0" (
set "file=%%a"
set "name=%%~na"
set "extension=%%~xa"
call set "name=%%name:%str1%=%str2%%%"
setlocal enabledelayedexpansion
ren "!file!" "!name!!extension!" 2>nul
endlocal
)
)
exit
```

# 游戏语音和音效压缩成aac

```batch
for %%i in (dir /s /b *.hca *.adx *.wav *.mp3 *.bcstm *.vag *.ogg *.wma *.m4a *.ss2 *.snd *.at3 *.at9 *.au *.aif *.bfstm *.msf *.mca *.brstm ) do ffmpeg -i %%i -c:a aac -b:a 32k "%%~ni.aac
pause
```
# 转换高质量视频

```batch
for %%i in (dir /s /b *.mkv *.avi *.usm *.wmv *.moflex *.m4v *.mpg *.webm *.bik *.3gp *.flv *.vob) do ffmpeg -i %%i -c:v libx265 -r 60 -crf 16 -preset fast -vf scale=2048:1080 -c:a aac -b:a 1536k "%%~ni.mp4
pause
```
> 在这个命令中使用了`libx265`也就是h265(HEVC)编码，如果你的电脑不支持该编码，可以尝试改为`libx264`使用常见的h264进行编码。
{.is-info}
