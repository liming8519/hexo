---
title: VirtualBox 操作命令随笔
tags:
  - tool
categories:
  - linux
date: 2019-04-16 01:00:00
---
> VirtualBox 操作命令随笔
<!-- more -->

　　
### 动态磁盘与固定磁盘转换
```
VBoxManage.exe list hdds
#动态转固定
VBoxManage.exe clonemedium disk "f:\s01.vdi" "f:\s02.vdi" -variant Fixed
#固定转动态
VBoxManage.exe clonemedium disk "f:\s01.vdi" "f:\s02.vdi" -variant Standard
```
### Oracle vm 克隆磁盘
```
VBoxManage.exe clonevdi D:\vm\node1\node1.vdi d:\vm\clone.vdi
```

### Oracle vm创建共享磁盘
```
VBoxManage.exe createhd --filename D:\vm\sharedisk\s01.vdi --size 30000 --format VDI -variant Fixed
VBoxManage.exe createhd --filename D:\vm\sharedisk\s02.vdi --size 30000 --format VDI -variant Fixed
VBoxManage.exe createhd --filename D:\vm\sharedisk\s03.vdi --size 30000 --format VDI -variant Fixed
```
### 附加磁盘到虚拟机
```
VBoxManage.exe storageattach linux1 --storagectl "SATA" --port 1 --device 0 --type hdd --medium D:\oraclevm\ocr.vdi --mtype shareable
```
### 修改磁盘为共享
```
VBoxManage.exe modifyhd D:\oraclevm\ocr.vdi --type shareable
```

