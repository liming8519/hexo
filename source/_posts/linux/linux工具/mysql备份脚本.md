---
title: mysql自动备份脚本
tags:
  - tool
categories:
  - linux
date: 2019-09-18 05:00:00
---
> mysql自动备份脚本
> <!-- more -->

###  脚本编写

```
[root@localhost ~]# mkdir /home/bak_data
[root@localhost ~]# mkdir /home/bak_data/log
[root@localhost ~]# vi bak_mysql.sh
#! /bin/bash
# name fengk
USER=root
PASSWORD=P@ssW0rd
DATABASE1=infoveriplatform_zhuozhou
BACKUP_DIR=/home/bak_data
LOGFILE=/home/bak_data/log/bak_data.log
DATE=`date +%Y%m%d-%H%M -d -3minute`
DUMPFILE1=$DATE-${DATABASE1}.sql
ARCHIVE1=$DUMPFILE1.gz
KEEPTIME=1
if [ ! -d $BACKUP_DIR ];
then
mkdir -p "$BACKUP_DIR"
fi

echo -e "\n" >> $LOGFILE
echo "-----------------------------------------------------" >> $LOGFILE
echo "BACKUP DATE:$DATE" >> $LOGFILE
echo "-----------------------------------------------------" >> $LOGFILE

cd $BACKUP_DIR
/home/mysql/bin/mysqldump --triggers --routines --events -h127.0.0.1 -P3306 -u$USER -p$PASSWORD --databases $DATABASE1 | gzip > $ARCHIVE1

if [[ $? == 0 ]];then
DELFILE=`find $BACKUP_DIR -type f -mtime +${KEEPTIME} -exec rm -fr {} \;`
echo "delete expire file " >> $LOGFILE
echo "-----------------------------------------------------" >> $LOGFILE
fi
[root@localhost ~]# chmod +x bak_mysql.sh
```

### 设置定时任务

```
[root@localhost ~]# vi /etc/corntab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
  30 00 *  *  * root  /root/bak_mysql.sh

[root@localhost ~]# systemctl restart crond
```