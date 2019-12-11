
---
title: 日志切割之logrotate
tags:
  - tool
categories:
  - linux
date: 2019-12-11 05:00:00
---
> 日志切割之logrotate
<!-- more -->

# 日志切割之logrotate
## logrotate配置文件位置
``` bash
/etc/logrotate.conf
/etc/logrotate.d/*
```
## logrotate配置实例
``` bash
$ cat /etc/logrotate.conf
# see "man logrotate" for details
# rotate log files weekly
weekly                  # 每周转存一次
# keep 4 weeks worth of backlogs
rotate 4                    # 保留四个日志备份文件
# create new (empty) log files after rotating old ones
create                      # rotate后,创建一个新的空文件
# use date as a suffix of the rotated file
dateext                     # 轮转的文件名字带有日期信息
# uncomment this if you want your log files compressed
#compress
# RPM packages drop log rotation information into this directory
include /etc/logrotate.d    # 此目录下的配置文件优先生效
# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {             # 指定/var/log/wtmp日志文件;
    monthly                 # 每月轮转一次,优先于全局设定的每周轮转一次;
    minsize 1M              # 日志文件大于1M才会去轮转;
    create 0664 root utmp   # 新日志文件的权限,属主,属组;
    rotate 1                # 保留一个日志备份,优先于全局设置的四个;
}
/var/log/btmp {             # 指定/var/log/btmp日志文件;
    missingok               # 如果日志丢失,不报错;
    monthly
    create 0600 root utmp
    rotate 1
}
# system-specific logs may be also be configured here.
# 系统特定的日志也可以在主文件中配置。


常用配置：
daily,weekly,monthly  # 转储周期分别是每天/每周/每月;
minsize 15M           # 日志体积大于此值时轮换(例如:100K,4M);
dateext               # 轮询的文件名字带有日期信息;
missingok             # 如果日志文件丢失,不要显示错误;
rotate 5              # 轮转存储中包含多少备份日志文件,0为无备份,以数字为准;
compress              # 通过gzip压缩转储以后的日志,以*.gz结尾;
nocompress            # 不需要压缩时,用这个参数;
delaycompress         # 延迟压缩,和compress一起使用时压缩所有日志,除当前和下一个最近的;
nodelaycompress       # 覆盖delaycompress选项，转储同时压缩;
copytruncate          # 用于还在打开中的日志文件,把当前日志备份并截断;
nocopytruncate        # 备份日志文件但是不截断;
create 644 www root   # 转储文件,使用指定的文件模式创建新的日志文件;
nocreate              # 不建立新的日志文件;
errors renwole@my.org # 专储时的错误信息发送到指定的Email地址;
ifempty               # 即使是空文件也转储，这个是logrotate的缺省选项；
notifempty            # 如果日志文件为空，则不转储;
mail renwole@my.org   # 把转储的日志文件发送到指定的E-mail地;
nomail                # 转储时不发送日志文件;
olddir /tmp           # 转储后的日志文件放入指定目录,必须和当前日志文件在同一个文件系统;
noolddir              # 转储后的日志文件和当前日志文件放在同一个目录下;
prerotate/endscript   # 在转储以前需要执行的命令可以放入这个对,这两个关键字必须单独成行;
postrotate/endscript  # 在转储以后需要执行的命令可以放入这个对,这两个关键字必须单独成行;
tabooext              # 不转储指定扩展名的文件,缺省扩展名：cfsaved,.disabled,.dpkg-dist等;
sharedscripts         # 共享脚本,让postrotate/endscript包含脚本只执行一次即可;
dateformat            # 配合dateext使用可以为切割后的日志加上YYYYMMDD格式的日期;
```