---
title:   win7 minifilter驱动程序开发
tags:
- tool
categories: 
- win 
date: 2019-12-02 02:00:00
---
>  win7 minifilter驱动程序开发
<!-- more -->

###  win7 minifilter驱动程序开发
```
注册表更改，可以输出测试信息：
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Debug Print Filter] 
"DEFAULT"=dword:0000000f
测试模式更改，管理员模式：
bcdedit /set testsigning on
bcdedit -set loadoptions DDISABLE_INTEGRITY_CHECKS

bcdedit.exe /set nointegritychecks on 禁止签名 

驱动调试时先右键安装ini文件，然后再使用Monitor安装sys文件，即可运行程序。

bat 安装驱动
@ copy %cd%\FFF.sys %windir%\system32\drivers\ /y
@bcdedit -set loadoptions DDISABLE_INTEGRITY_CHECKS
@bcdedit.exe /set nointegritychecks on
@sc create FFF binpath= %cd%\\FFF.sys type= kernel start= auto error= ignore DisplayName= FFF
@reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\FFF\Instances" /v "DefaultInstance" /d "FFF Instance" /t REG_SZ /f
@reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\FFF\Instances\FFF Instance" /v "Altitude" /d "370030" /t REG_SZ /f
@reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\FFF\Instances\FFF Instance" /v "Flags" /d "00000000" /t REG_DWORD /f
@net start FFF

bat卸载驱动
@net stop FFF
@sc delete FFF
```