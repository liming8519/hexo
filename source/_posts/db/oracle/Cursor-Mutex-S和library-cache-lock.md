---
title: Cursor mutex S和library cache lock
tags:
  - oracle
categories:
  - db
date: 2019-03-31 02:00:00

---


> Cursor: Mutex S和library cache lock
> <!-- more -->

>实际在版本10g中就引入了该Cursor Obsolescence游标废弃特性，当时child cursor 的总数阀值是1024，但是这个阀值在11g中被移除了，这导致出现一个父游标下大量child cursor即high version count的发生；由此引发了一系列的版本11.2.0.3之前的cursor sharing 性能问题，主要症状是版本11.2.0.1和11.2.0.2上出现大量的Cursor: Mutex S 和 library cache lock等待事件。

>增强补丁Enhancement patch《Bug 10187168 – Enhancement to obsolete parent cursors if VERSION_COUNT exceeds a threshold》就该问题引入了新的隐藏参数_cursor_obsolete_threshold(Number of cursors per parent before obsoletion.)，该”_cursor_obsolete_threshold”参数用以指定子游标总数阀值，若一个父游标的child cursor count<=>version count高于”_cursor_obsolete_threshold”，则触发Cursor Obsolescence游标废弃特性。**注意版本11.2.0.3中默认就有”_cursor_obsolete_threshold”了，而且默认值为100。**


>对于版本11.1.0.7、11.2.0.1和11.2.0.2则都有该Bug 10187168的bug backport存在，从2011年5月开始就有相关针对的one-off backport补丁存在。 但是这些one-off backport补丁不使用”_cursor_obsolete_threshold”参数。在版本11.1.0.7、11.2.0.1和11.2.0.2上需要设置合适的”_cursor_features_enabled”(默认值为2)参数，并设置必要的106001 event，该event的level值即是child cursor count的阀值，必须设置该106001事件后该特性才生效。但是请注意 “_cursor_features_enabled”参数需要重启实例方能生效。而”_cursor_obsolete_threshold”参数和106001 event则可以在线启用、禁用。对于不同的版本而言，一般推荐打上最新的PSU补丁，并根据补丁的README提示或者咨询Oracle Support获得关于该版本上Cursor Obsolescence问题的信息：
```
#针对不同版本设置 “_cursor_features_enabled”+106001 event的方法。
#注意重启实例
#版本11.1.0.7
SQL> alter system set "_cursor_features_enabled"=18 scope=spfile;
SQL> alter system set event='106001 trace name context forever,level 1024' scope=spfile;
#版本11.2.0.1
SQL> alter system set "_cursor_features_enabled"=34 scope=spfile;
SQL> alter system set event='106001 trace name context forever,level 1024' scope=spfile;
#版本11.2.0.2
SQL> alter system set "_cursor_features_enabled"=1026 scope=spfile;
SQL> alter system set event='106001 trace name context forever,level 1024' scope=spfile;
```

```
11gR2 Experience -> If using cursor_sharing = “FORCE” or “SIMILAR”
　　1) ORA-600 errors as workload increases [kkspsc0: basehd]
　　or [kglLockOwnersListAppend-ovf] - applied patches to address
　　2) AWR showing -> cursor: mutex S and library cache lock
1. Download and apply the 11.2.0.2.3PSU Patch 11724916
2. Enable event 106001 to address Bug 10187168.
　　To enable the fix "_cursor_features_enabled" needs to be set
　　3) Oracle 11.2.0.2.2 PSU (Patch Set Update) includes new parameters that you can tweak
　　based on workload characteristics. Even more fixes have been added
　　Note: 10411618 - Enhancement to add different "Mutex" wait schemes [ID 10411618.8]
　　4) 11.2.0.3 Has many Mutex enhancement’s
　　106001 level
　　The level is used to specify the maximum number of child cursors
　　that a parent can have before we obsolete
　　the parent cursor and create a new parent.
　　Doing the above can help reduce mutex waits,
　　memory consumption and other side effects seen when we see
　　many child cursors for a given parent.
```