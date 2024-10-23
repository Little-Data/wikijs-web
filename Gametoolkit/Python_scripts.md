---
title: Python处理脚本
description: 更好的解包辅助
published: 1
date: 2024-10-23T14:27:09.842Z
tags: python, 游戏工具, 脚本
editor: markdown
dateCreated: 2024-10-23T14:27:09.842Z
---

# 初次使用？
在使用这些脚本之前，请确认已经安装了Python
如果没有，请到官网下载。由于为外网，网页可能打不开
- [Download Python](https://www.python.org/downloads)
{.links-list}

自行复制代码，再粘贴到一个文本文件内，然后把`.txt`后缀改为`.py`
也可以到[这里](https://wp.little-data.eu.org/cr2/%E8%AE%A1%E7%AE%97%E6%9C%BA%E6%96%87%E4%BB%B6%E5%B1%9E%E6%80%A7%E6%B1%87%E6%80%BB)直接下载源码文件。
# 3dstool自动解包
3ds解包脚本，使用任天堂toolbox有时候仍然解不开一些3ds游戏，不得不再次使用3dstool去解，由于它这个命令解包太复杂，所以制作了个脚本，可以很轻松搞定3ds游戏的解包流程。
使用之前，先安装依赖：
```python
pip install tkinterdnd2 -i https://pypi.tuna.tsinghua.edu.cn/simple
```
该程序源码：
```python
import tkinter as tk
from tkinter import filedialog
import subprocess
import os

def extract_3ds():
    # 获取用户选择的文件
    file_path = filedialog.askopenfilename(filetypes=[("3DS files", "*.3ds")])
    if not file_path:
        return
    
    # 获取文件所在目录和文件名
    dir_path = os.path.dirname(file_path)
    file_name = os.path.splitext(os.path.basename(file_path))[0]

    # 创建与ROM同名的文件夹
    rom_folder = os.path.join(dir_path, file_name)
    os.makedirs(rom_folder, exist_ok=True)
    
    # 执行3dstool命令提取cci
    cci_command = f'3dstool -xvt0f cci {rom_folder}\\0.cxi "{file_path}" --header {rom_folder}\\ncsdheader.bin'
    cci_process = subprocess.run(cci_command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
    print(cci_process.stdout)
    
    # 创建cxi0子文件夹
    cxi0_folder = os.path.join(rom_folder, 'cxi0')
    os.makedirs(cxi0_folder, exist_ok=True)

    # 执行3dstool命令提取cxi文件内容
    cxi_command = f'3dstool -xvtf cxi {rom_folder}\\0.cxi --header {cxi0_folder}\\ncchheader.bin --exh {cxi0_folder}\\exh.bin --plain {cxi0_folder}\\plain.bin --exefs {cxi0_folder}\\exefs.bin --romfs {cxi0_folder}\\romfs.bin'
    cxi_process = subprocess.run(cxi_command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
    print(cxi_process.stdout)

    # 执行3dstool命令解压romfs.bin
    romfs_command = f'3dstool -xvtf romfs {cxi0_folder}\\romfs.bin --romfs-dir {cxi0_folder}\\romfs'
    romfs_process = subprocess.run(romfs_command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
    print(romfs_process.stdout)

    print(f"Extraction complete for {file_name}")

def on_button_click():
    extract_3ds()

# 创建主窗口
root = tk.Tk()
root.title("3ds游戏解包")
root.geometry("600x300")
root.resizable(True, True)

# 创建按钮并绑定点击事件
extract_button = tk.Button(root, text="Extract 3DS File", command=on_button_click)
extract_button.pack(pady=20)

# 启动事件循环
root.mainloop()
```
然后双击打开脚本，选择你要解的3ds游戏rom，就ok了，它会自动创建个跟rom名字一样的文件夹，然后剩下就全程自动解里面的`cxi`和`romfs.bin`，并提取`romfs.bin`里面的所有资源。

# 提取图片
二进制数据读取，批量提取游戏里面存储的png、jpg、dds和webp图片，任何没有使用特殊加密的游戏都可以解出来。
分为四个文件：
`ddsextract.py`
`jpgextract.py`
`pngextract.py`
`webpextract.py`
视频讲解：
- [读取二进制数据，自制脚本提取图片*bilibili*](https://www.bilibili.com/video/BV1dU22YUE6X)
{.links-list}

`ddsextract.py`

```python
import os

def extract_dds_content(file_path, directory_path, start_sequence, end_sequence):
    with open(file_path, 'rb') as file:
        content = file.read()
        count = 0
        start_index = 0  # 从文件开头开始

        while start_index < len(content):
            # 查找文件头
            start_index = content.find(start_sequence, start_index)
            if start_index == -1:
                print(f"No more start sequences found in {file_path}")
                break

            # 查找下一个文件头
            next_start_index = content.find(start_sequence, start_index + len(start_sequence))
            if next_start_index == -1:
                end_index = len(content)
            else:
                end_index = next_start_index

            # 提取从当前文件头到下一个文件头之前的内容
            extracted_data = content[start_index:end_index]
            # 生成新的文件名
            new_filename = f"{os.path.splitext(os.path.basename(file_path))[0]}_extracted_{count}.dds"
            new_filepath = os.path.join(directory_path, new_filename)
            # 保存到新文件
            with open(new_filepath, 'wb') as new_file:
                new_file.write(extracted_data)
            print(f"Extracted content saved as: {new_filepath}")
            count += 1

            # 更新内容，继续查找
            start_index = next_start_index

def main():
    directory_path = input("请输入要处理的文件夹路径: ")
    if not os.path.isdir(directory_path):
        print(f"错误: {directory_path} 不是一个有效的目录。")
        return

    start_sequence = b'\x44\x44\x53\x20\x7C'  # DDS file header
    end_sequence = b'\x44\x44\x53\x20\x7C'  # Next DDS file header

    for root, dirs, files in os.walk(directory_path):
        for file in files:
            file_path = os.path.join(root, file)
            print(f"Processing file: {file_path}")
            extract_dds_content(file_path, directory_path, start_sequence, end_sequence)

if __name__ == "__main__":
    main()
```

`jpgextract.py`

```python
import os

def is_valid_jpg(data):
    # JPEG文件的开始标记
    start_marker = b'\xFF\xD8\xFF\xE0'
    # JPEG文件的结束标记
    end_marker = b'\xFF\xD9'
    # 数据块标记（JFIF或EXIF）
    block_markers = [b'JFIF', b'Exif']

    # 检查JPEG文件的开始标记
    if not data.startswith(start_marker):
        return None

    # 检查数据块开始标记
    for block_marker in block_markers:
        block_index = data.find(block_marker)
        if block_index != -1:
            # 检查文件尾标记
            end_index = data.find(end_marker, block_index)
            if end_index != -1:
                # 提取JPEG数据
                return data[:end_index + len(end_marker)]
    return None

def find_jpg_in_file(file_path):
    with open(file_path, 'rb') as file:
        file_content = file.read()
        jpg_start = file_content.find(b'\xFF\xD8\xFF\xE0')
        jpgs = []
        while jpg_start != -1:
            # 从文件头开始查找有效的JPEG数据
            jpg_data = is_valid_jpg(file_content[jpg_start:])
            if jpg_data:
                jpgs.append(jpg_data)
            # 移动到下一个可能的文件头位置
            jpg_start = file_content.find(b'\xFF\xD8\xFF\xE0', jpg_start + len(b'\xFF\xD8\xFF\xE0'))
        return jpgs

def extract_jpgs(directory_path):
    total_jpgs_extracted = 0
    for root, dirs, files in os.walk(directory_path):
        for file in files:
            file_path = os.path.join(root, file)
            # Skip Python files and files containing 'disabled'
            if file.endswith('.py') or 'disabled' in file.lower():
                continue

            jpgs = find_jpg_in_file(file_path)
            for index, jpg_data in enumerate(jpgs):
                base_filename, _ = os.path.splitext(file)
                extracted_filename = f'{base_filename}_extracted_{index}.jpg'
                extracted_path = os.path.join(root, extracted_filename)
                with open(extracted_path, 'wb') as jpg_file:
                    jpg_file.write(jpg_data)
                total_jpgs_extracted += 1

    return total_jpgs_extracted

def main():
    directory_path = input("请输入要处理的文件夹路径: ")
    if not os.path.isdir(directory_path):
        print(f"错误: {directory_path} 不是一个有效的目录。")
        return

    total_jpgs_extracted = extract_jpgs(directory_path)
    print(f"Total jpgs extracted: {total_jpgs_extracted}")

if __name__ == "__main__":
    main()
```

`pngextract.py`

```python
import os

def extract_content(file_path, directory_path, start_sequence, block_marker, end_sequence):
    with open(file_path, 'rb') as file:
        content = file.read()
        start_index = content.find(start_sequence)
        if start_index == -1:
            print(f"No start sequence found in {file_path}")
            return
        
        # 从第一个文件头开始读取
        content = content[start_index:]
        count = 0
        while True:
            start_index = content.find(start_sequence)
            if start_index == -1:
                print(f"No more start sequences found in {file_path}")
                break

            # 从找到的字节序列开始，直到下一个字节序列结束
            next_start_index = content.find(start_sequence, start_index + len(start_sequence))
            if next_start_index == -1:
                # 如果没有找到下一个文件头，则提取剩余的内容
                end_index = len(content)
            else:
                # 提取从文件头到下一个文件头之前的内容
                end_index = next_start_index

            # 提取内容
            extracted_data = content[start_index:end_index]
            # 检查是否为有效PNG（即是否包含数据块标记）
            if block_marker in extracted_data:
                # 生成新的文件名
                new_filename = f"{os.path.splitext(os.path.basename(file_path))[0]}_extracted_{count}.png"
                new_filepath = os.path.join(directory_path, new_filename)
                # 保存到新文件
                with open(new_filepath, 'wb') as new_file:
                    new_file.write(extracted_data)
                print(f"Extracted content saved as: {new_filepath}")
                count += 1
            else:
                print(f"Invalid PNG data found in {file_path}, skipping...")

            # 更新内容，继续查找
            if next_start_index == -1:
                break
            content = content[next_start_index:]

def main():
    directory_path = input("请输入要处理的文件夹路径: ")
    if not os.path.isdir(directory_path):
        print(f"错误: {directory_path} 不是一个有效的目录。")
        return

    # 定义要查找的字节序列
    start_sequence = b'\x89\x50\x4E\x47'
    block_marker = b'\x49\x48\x44\x52'
    end_sequence = b'\x89\x50\x4E\x47'

    for root, dirs, files in os.walk(directory_path):
        for file in files:
            file_path = os.path.join(root, file)
            print(f"Processing file: {file_path}")
            extract_content(file_path, directory_path, start_sequence, block_marker, end_sequence)

if __name__ == "__main__":
    main()
```

`webpextract.py`

```python
import os
import struct

def extract_webp_data(file_content):
    riff_header = b'\x52\x49\x46\x46'
    webp_header = b'\x57\x45\x42\x50'

    # 跳过第一个文件头之前的内容
    file_content = file_content[file_content.find(riff_header):]
    
    webp_data_start = file_content.find(riff_header)
    while webp_data_start != -1:
        # 检查文件大小
        file_size = struct.unpack('<I', file_content[webp_data_start + 4:webp_data_start + 8])[0]
        # 检查 WEBP 标记
        if file_content[webp_data_start + 8:webp_data_start + 12] == webp_header:
            # 提取 WebP 图像数据
            webp_data = file_content[webp_data_start:webp_data_start + file_size + 8]
            yield webp_data
        # 移动到下一个 RIFF 标记的位置
        webp_data_start = file_content.find(riff_header, webp_data_start + file_size + 8)

def extract_webps_from_file(file_path):
    with open(file_path, 'rb') as file:
        file_content = file.read()

    webp_data_generator = extract_webp_data(file_content)
    count = 0
    for webp_data in webp_data_generator:
        # 保存 WebP 数据到文件
        base_filename, _ = os.path.splitext(os.path.basename(file_path))
        extracted_filename = f"{base_filename}_{count}.webp"
        extracted_path = os.path.join(os.path.dirname(file_path), extracted_filename)
        with open(extracted_path, 'wb') as output_file:
            output_file.write(webp_data)
        print(f"Extracted content saved as: {extracted_path}")
        count += 1

def extract_webps(directory_path):
    for root, dirs, files in os.walk(directory_path):
        for file in files:
            # 跳过.py文件
            if file.endswith('.py'):
                continue
            file_path = os.path.join(root, file)
            extract_webps_from_file(file_path)

def main():
    directory_path = input("请输入要处理的文件夹路径: ")
    if not os.path.isdir(directory_path):
        print(f"错误: {directory_path} 不是一个有效的目录。")
        return

    extract_webps(directory_path)
    print("WebP 文件提取完成。")

if __name__ == "__main__":
    main()
```