---
title: 生成awr报告
tags:
  - oracle
categories:
  - db
date: 2019-03-31 01:00:00

---



> 生成awr报告
<!-- more -->

1. sqlplus下执行：
   @?/rdbms/admin/awrrpt.sql
   或者@$ORACLE_HOME/rdbms/admin/awrrpt.sql
2. 输入要生成报告的文件格式
   Type Specified:  html
3. 输入要生成报告相隔的天数
   Enter value for num_days: 1
4. 输入相隔的快照之间的Snap Id开始号和结束号
   Specify the Begin and End Snapshot Ids
   Enter value for begin_snap: 101
   Enter value for end_snap: 102
   End   Snapshot Id specified: 102
5. 输入生成报告的名字：
Enter value for report_name: awr.html