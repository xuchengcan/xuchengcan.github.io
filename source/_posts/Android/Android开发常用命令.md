---
title: Android 开发常用命令
categories:
  - Android
comments: true
date: 2018-08-08 13:46:09
updated: 2018-08-08 13:46:09
tags: 
  - Android 
  - 应用安全防护和逆向分析
keywords:
description:
---

# Android 开发常用命令整理

参考
- 《Android 应用安全防护和逆向分析》

<!-- more -->

### adb shell 命令

> adb shell getprop

查看系统属性

```shell
vivo： getprop ro.build.version.opporom       获取系统版本号
miui： getprop ro.miui.ui.version.name
```

> adb shell dumpsys activity top

查看当前应用的 activity 信息

> adb shell dumpsys package [package]

查看指定包名应用的详细信息（相当于应用的 AndroidManifest.xml）

> adb shell dumpsys meminfo [pname/id]

查看指定进程名或者进程 id 的内存信息

> adb shell dumpsys dbinfo [package]

可以查看指定包名应用的数据库存储信息（包括存储的 SQL 语句）

> adb shell screencap -p [路径]

截屏到指定路径

```shell
adb shell screencap -p /sdcard/temp.png
adb pull /sdcard/temp.png D:\
start D:\temp.png
```

> adb forward [(远程端) 协议：端口号] [(设备端) 协议：端口号]

设备的端口转发，该命令在 IDA 调试中非常有用

    adb forward tcp:8700 jdwp:1786

> adb jdwp

查看设备中可以被调试的应用的进程号

### adb shell 中的命令

> run-as [package name]

在非 root 设备中查看指定 debug 模式的包名应用沙盒数据

> ps
> ps | grep 过滤内容
> ps -t [pid] 查看 pid 对应的进程信息

> pm clear [package name]

清楚指定包名应用的数据

> am start

启动一个应用

```shell
am start -n [package]/[package].[activity]
```

注意：可以用 debug 方式启动应用 ( am start -D -n ... ）。特别在反编译调试应用的时候，可能需要 debug 方式启动应用。

> am startservice

启动一个服务

```shell
am startservice -n [package]/[package].[service]
```

> am broadcast

发送一个广播

```shell
am broadcast -a [broadcast]
```

> netcfg

查看设备 ip 地址

> netstat

查看设备的端口号信息

> app_process

运行 java 代码

这个命令主要用于 Android 中一些特殊的开发场景中，想启动一个 jar 包，不过这个 jar 包有要求：需要 dx 命令把 dex 文件转化成 jar 包功能，实际上它不是一个正常的 jar 包了，而是一个包含了 class.dex 文件的压缩文件了。

> dalvikvm

运行一个 dex 文件

有时候为了测试一个 dex 文件功能可以用到这个命令，与上面的命令有很大的相似之处，只是运行的文件不一样。

> top

查看当前应用的 CPU 信息

```shell
tap [-n/-m/-d/-s/-t]
-m  最多显示多少个进程
-n  刷新次数
-d  刷新间隔时间
-s  按哪列排序
-t  显示线程信息而不是进程

top -d 1 -m 10
```

这个命令在分析应用性能的时候非常有用，可以用 grep 过滤想要分析的应用信息，查看它的当前 CPU 使用率。

> getprop

查看系统属性值

getprop ro.debuggable

root 设备之后，可以通过修改这些系统属性。比如 debug 开关，让所有应用都处于可调试状态。

### 操作 apk 命令

> aapt

查看 apk 中的信息以及编辑 apk 程序包

```shell
aapt dump xmltree [.apk] [.xml]
```

> dexdump [dex 文件路径]

查看一个 dex 文件的详细信息

### 进程命令

> cat /proc/[pid]/maps

查看当前进程的内存映射信息，比如加载了哪些 so 文件， dex 文件等。

> cat /proc/[pid]/status

查看当前进程的状态信息，比如 TracerPid 。

### adb 获取手机 apk

- 列出手机包名：adb shell pm list package

- 获取 apk 安装路径：adb shell pm path ‘package name’
adb shell pm path com.google.android.youtube
package:/data/app/com.google.android.youtube-1/base.apk

- 使用 pull 命令获取apk：adb pull ‘package-path’ ‘new.apk’
adb pull /data/app/com.google.android.youtube-1/base.apk youtube.apk
/data/app/com.google.android.youtube-1/base.apk: 1 file pulled. 2.7 MB/s (21504340 bytes in 7.724s)

- 总结： adb shell pm 命令获取安装包路径， adb pull 命令拿到安装包。

### adb wifi 调试

- adb tcpip [端口号（随便写个大点的比如：5555）]

- adb connect [手机 ip]

- adb usb （换回usb模式 ）