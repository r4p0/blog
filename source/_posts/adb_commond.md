---
title: adb 命令笔记
date: 2024-08-05 10:09:42
updated: 2024-08-14 15:29:53
tags: 
- adb
- shell
comments: true
---

## adb 命令
- adb devices
列出当前链接adb的设备
```shell
$ adb devices
List of devices attached
emulator-5558   device
```

- adb pair {ip}:{port}
配对指定ip和端口的adb设备，使用connect前需要先执行pair
手机端需要打开 开发者选项 / 无线调试 / 使用配对码配对设备
```shell
$ adb pair 172.16.12.140:38617
Enter pairing code: ******
Successfully paired to 172.16.12.140:38617 [guid=adb-********-******]
```

- adb connect {ip}:{port}
连接指定ip和端口的adb设备
```shell
$ adb connect 172.16.12.140:43357
connected to 172.16.12.140:43357
```

- adb disconnect
断开所有连接adb的设备
```shell
$ adb disconnect
disconnected everything
```

- adb -s {device id} {adb commond}
指定要操作的adb设备

- adb shell
进入设备的shell命令行
```shell
$ adb shell
marlin:/ $
```

- adb shell {commond}
进入设备进入shell并执行命令{commond}

- adb install-multiple {path1} {path2} ... {pathN}
安装多个APK可以用于手动安装xapk文件（低版本ADB可能不支持）

## android shell 命令

- pm list packages
显示所有包名
```shell
$ adb shell
marlin:/ $ pm list packages
package:com.android.providers.telephony
package:com.android.providers.calendar
package:com.android.providers.media
...
```

- pm path {pachage name}
显示指定包名安装包的路径（xapk会有多个路径）
```shell
$ adb shell
marlin:/ $ pm path com.android.providers.media
package:/system/priv-app/MediaProvider/MediaProvider.apk
```
