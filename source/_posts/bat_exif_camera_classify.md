---
title: BAT 脚本：根据 EXIF 信息分类照片
date: 2025-12-16 17:33:53
tags:
- bat
- exif
---
# BAT 脚本：根据 EXIF 信息分类照片

本脚本会扫描当前目录下的 JPG/HEIF 照片，根据 EXIF 信息将它们分类到不同的目录：

```bat
@echo off
setlocal enabledelayedexpansion

:: 设置支持的图片扩展名
set "extensions=.jpg .jpeg .heif .heic"

:: 创建必要的目录
if not exist "Clouds" mkdir "Clouds"
if not exist "Unknown" mkdir "Unknown"

:: 遍历当前目录下的所有图片文件
for %%f in (*) do (
    set "file=%%f"
    set "extension=%%~xf"
    
    :: 检查文件扩展名是否在支持的列表中
    echo %extensions% | find /i "!extension!" > nul
    if !errorlevel! equ 0 (
        echo 处理文件: !file!
        
        :: 使用 PowerShell 获取 EXIF 信息
        for /f "delims=" %%a in ('powershell -command "[Console]::OutputEncoding = [System.Text.Encoding]::UTF8; $shell = New-Object -COMObject Shell.Application; $folder = $shell.Namespace((Get-Item '!file!').DirectoryName); $file = $folder.ParseName((Get-Item '!file!').Name); $comment = $folder.GetDetailsOf($file, 24); $cameraModel = $folder.GetDetailsOf($file, 271); $cameraModel = $cameraModel -replace '[\\u200E\\u202A-\\u202E]', ''; if ($comment) { 'Clouds' } elseif (-not $cameraModel -or $cameraModel -eq '') { 'Unknown' } else { $cameraModel }"') do (
            set "target_dir=%%a"
        )
        
        :: 清理目录名中的非法字符
        set "clean_dir=!target_dir!"
        set "clean_dir=!clean_dir:\=!"
        set "clean_dir=!clean_dir:/=!"
        set "clean_dir=!clean_dir::=!"
        set "clean_dir=!clean_dir:?=!"
        set "clean_dir=!clean_dir:"=!"
        set "clean_dir=!clean_dir:<=!"
        set "clean_dir=!clean_dir:>=!"
        set "clean_dir=!clean_dir:|=!"
        set "clean_dir=!clean_dir: =_!"
        
        :: 创建目标目录（如果不存在）
        if not exist "!clean_dir!" mkdir "!clean_dir!"
        
        :: 移动文件
        echo 移动 "!file!" 到 "!clean_dir!\"
        move "!file!" "!clean_dir!\" > nul
    )
)

echo 文件分类完成!
pause
```

## 脚本说明

1. **支持的格式**：脚本处理 JPG、JPEG、HEIF 和 HEIC 格式的文件

2. **分类规则**：
   - 如果照片有备注认定为小米云相册（属性24），则移动到 `Clouds` 目录
   - 如果没有相机型号（属性271），则移动到 `Unknown` 目录
   - 如果有相机型号，则移动到以相机型号命名的目录

3. **特殊处理**：
   - 清理 EXIF 信息中的 Unicode 控制字符（U+200E, U+202A-U+202E）
   - 清理目录名中的非法字符（\/:*?"<>|等）

4. **EXIF 属性索引**：
   - 24 = 备注/注释
   - 271 = 相机型号

## 使用说明

1. 将脚本保存为 `classify_photos.bat`
2. 将脚本放在包含照片的目录中
3. 双击运行脚本
4. 脚本会自动创建必要的目录并移动文件

## 注意事项

1. 脚本不会修改或删除原始文件，只是移动它们
2. 如果文件名包含特殊字符，可能需要额外处理
3. 对于大量文件，处理可能需要一些时间
4. 如果相机型号包含特殊字符，脚本会自动替换为下划线