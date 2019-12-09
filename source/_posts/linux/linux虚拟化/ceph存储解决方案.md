---
title:   ceph存储解决方案
tags:
- tool
categories: 
- linux 
date: 2019-11-29 02:00:00
---
>  ceph存储解决方案
<!-- more -->

###  ceph存储解决方案

```
下载http://eu.ceph.com/rpm-luminous/el7/x86_64/
创建私有yum源，比较麻烦，把测试环境的包拷贝到百度网盘以备后用。


免密：
ssh-keygen
ssh-copy-id 目标机器ip

chrony 时钟同步见之前文档

vi /etc/hosts
10.10.10.1 ceph1
10.10.10.2 ceph2
10.10.10.3 ceph3


[root@managementa ceph]# yum install ceph-deploy
[root@managementa ceph]# yum install ceph


[root@managementa ceph]# ceph-deploy new ceph1 创建配置文件
[root@managementa ceph]# mkdir /etc/ceph
[root@managementa ceph]# cd /etc/ceph
[root@managementa ceph]# ceph-deploy install --release ceph1 ceph2 ceph3  这一步被yum install ceph代替
[root@managementa ceph]# ceph-deploy mon create-initial
[root@managementa ceph]# ceph status
[root@managementa ceph]# ceph-deploy disk list ceph1
root@ceph1 ceph]# ceph-deploy disk zap ceph1 /dev/sdb  /dev/sdc /dev/sdd
[root@ceph1 ceph]# ceph-deploy osd create ceph1 --data /dev/sdb 

[root@ceph1 ceph]# ceph status

分发授权
[root@ceph1 ceph]# ceph-deploy admin ceph1 ceph2 ceph3

创建mgr
[root@ceph1 ceph]# ceph-deploy mgr create ceph1

指定网络（只在主节点就行，其他节点的conf文件会跟随变化，主节点拷贝过去的）
[root@ceph1 ceph]# vi /etc/ceph/ceph.conf 
public_network  = 10.10.10.0/24
cluster_network = 10.10.10.0/24
```

### 其他节点加入集群

```
[root@ceph1 ceph]# ceph-deploy disk list ceph2
[root@ceph1 ceph]# ceph-deploy disk list ceph3
[root@ceph1 ceph]# ceph-deploy disk zap ceph2 /dev/sdb /dev/sdc /dev/sdd
[root@ceph1 ceph]# ceph-deploy disk zap ceph3 /dev/sdb /dev/sdc /dev/sdd
[root@ceph1 ceph]# ceph-deploy osd create ceph2 --data /dev/sdb
[root@ceph1 ceph]# ceph-deploy osd create ceph3 --data /dev/sdb
[root@ceph2 ceph]# ceph status

[root@ceph1 ceph]# ceph-deploy mon create ceph2    
[root@ceph1 ceph]# ceph-deploy --overwrite-conf mon create ceph3
[root@ceph1 ceph]# ceph -s
```

### 操作随笔

```
[root@ceph1 ~]# ceph osd pool create rbd 16
[root@ceph1 ~]# rbd create myd1 --size 1024
[root@ceph1 ~]# rbd ls
myd1
[root@ceph1 ~]# rbd feature disable myd1 exclusive-lock,object-map,fast-diff,deep-flatten
[root@ceph1 ~]# rbd info --image myd1
rbd image 'myd1':
        size 1GiB in 256 objects
        order 22 (4MiB objects)
        block_name_prefix: rbd_data.5f0a6b8b4567
        format: 2
        features: layering
        flags: 
        create_timestamp: Thu Dec  5 07:40:30 2019
[root@ceph1 ~]# ceph --show-config | grep rbd | grep feature
rbd_default_features = 61 /*设置为1时仅支持layering，可以在/etc/ceph/ceph.conf中配置*/
[root@ceph1 ~]# rbd map --image myd1
/dev/rbd0
可以使用这个磁盘了
[root@ceph1 ~]# rbd showmapped
id pool image snap device    
0  rbd  myd1  -    /dev/rbd0 
[root@ceph1 ~]# fdisk -l /dev/rbd0
[root@ceph1 ~]# mkfs.xfs /dev/rbd0
[root@ceph1 ~]# mkdir myf
[root@ceph1 ~]# mount /dev/rbd0 myf
[root@ceph1 ~]# cd myf/
[root@ceph1 myf]# dd if=/dev/zero of=file1 count=10 bs=1M

调整大小
[root@ceph1 myf]# rbd resize rbd/myd1 --size 2048
[root@ceph1 myf]# xfs_growfs -d /dev/rbd0
[root@ceph1 myf]# df -T
Filesystem          Type     1K-blocks    Used Available Use% Mounted on
/dev/mapper/cl-root xfs       52403200 2266244  50136956   5% /
devtmpfs            devtmpfs    488980       0    488980   0% /dev
tmpfs               tmpfs       499968       0    499968   0% /dev/shm
tmpfs               tmpfs       499968    6856    493112   2% /run
tmpfs               tmpfs       499968       0    499968   0% /sys/fs/cgroup
/dev/sda1           xfs        1038336  141676    896660  14% /boot
/dev/mapper/cl-home xfs       49250820   32944  49217876   1% /home
tmpfs               tmpfs       499968      24    499944   1% /var/lib/ceph/osd/ceph-0
tmpfs               tmpfs        99996       0     99996   0% /run/user/0
/dev/rbd0           xfs        2086912   43600   2043312   3% /root/myf

快照
[root@ceph1 myf]# echo 1234 >snaptett_file
[root@ceph1 myf]# cat snaptett_file 
1234
[root@ceph1 myf]# rbd snap create rbd/myd1@snap1
[root@ceph1 myf]# rbd snap ls rbd/myd1
SNAPID NAME  SIZE TIMESTAMP                
     4 snap1 2GiB Thu Dec  5 08:36:46 2019 
[root@ceph1 myf]# 
[root@ceph1 myf]# rm -fr snaptett_file 
[root@ceph1 myf]# ls
file1
[root@ceph1 myf]#
[root@ceph1 myf]# rbd snap rollback rbd/myd1@snap1
Rolling back to snapshot: 100% complete...done.
[root@ceph1 ~]# fuser -mv -k myf
[root@ceph1 ~]# umount myf（umount之后再rollback）
[root@ceph1 ~]# rbd snap rollback rbd/myd1@snap1
Rolling back to snapshot: 100% complete...done.
[root@ceph1 ~]# mount /dev/rbd0 myf

[root@ceph1 myf]# rbd snap rm rbd/myd1@snap1
Removing snap: 100% complete...done.
[root@ceph1 myf]# rbd snap purge rbd/myd1
[root@ceph1 ~]# umount myf

[root@ceph1 ~]# rbd status myd1
Watchers:
        watcher=10.10.10.1:0/2334948862 client.24378 cookie=1
[root@ceph1 ~]# rbd info myd1
rbd image 'myd1':
        size 2GiB in 512 objects
        order 22 (4MiB objects)
        block_name_prefix: rbd_data.5f0a6b8b4567
        format: 2
        features: layering
        flags: 
        create_timestamp: Thu Dec  5 07:40:30 2019
[root@ceph1 ~]# rados -p rbd listwatchers rbd_header.5f0a6b8b4567
watcher=10.10.10.1:0/2334948862 client.24378 cookie=1
[root@ceph1 ~]# ceph osd blacklist add 10.10.10.1:0/2334948862
blacklisting 10.10.10.1:0/2334948862 until 2019-12-05 10:06:25.192335 (3600 sec)
[root@ceph1 ~]# rados -p rbd listwatchers rbd_header.5f0a6b8b4567
[root@ceph1 ~]# 
[root@ceph1 ~]# rbd rm myd1 -p rbd
Removing image: 100% complete...done.
[root@ceph1 ~]# 
[root@ceph1 ~]# ceph osd blacklist clear
ceph osd blacklist rm 10.10.10.1:0/2334948862








[root@ceph1 ~]# ceph health detail
HEALTH_WARN application not enabled on 1 pool(s)
POOL_APP_NOT_ENABLED application not enabled on 1 pool(s)
    application not enabled on pool 'rbd'
    use 'ceph osd pool application enable <pool-name> <app-name>', where <app-name> is 'cephfs', 'rbd', 'rgw', or freeform for custom applications.
[root@ceph1 ~]# ceph osd pool application enable rbd rbd
```

### 写时复制 cow

```
[root@ceph1 ~]# rbd create myd1 --size 1024 --image-format 2
rbd image 'myd1':
        size 1GiB in 256 objects
        order 22 (4MiB objects)
        block_name_prefix: rbd_data.5fa06b8b4567
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        flags: 
        create_timestamp: Thu Dec  5 09:13:28 2019
[root@ceph1 ~]# rbd snap create rbd/myd1@snapshot_for_clone
[root@ceph1 ~]# rbd snap protect rbd/myd1@snapshot_for_clone 
[root@ceph1 ~]# rbd clone rbd/myd1@snapshot_for_clone rbd/myd2_c
[root@ceph1 ~]# rbd -p rbd --image myd2_c info

扁平化，不依赖任何父节点
[root@ceph1 ~]# rbd flatten rbd/myd2_c
[root@ceph1 ~]# rbd -p rbd --image myd2_c info
rbd image 'myd2_c':
        size 1GiB in 256 objects
        order 22 (4MiB objects)
        block_name_prefix: rbd_data.5fac6b8b4567
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        flags: 
        create_timestamp: Thu Dec  5 09:16:59 2019
[root@ceph1 ~]# rbd snap unprotect rbd/myd1@snapshot_for_clone
[root@ceph1 ~]# rbd snap rm rbd/myd1@snapshot_for_clone
Removing snap: 100% complete...done.
```
### mds操作

```

[root@ceph1 ~]# systemctl stop ceph-mds.target
[root@ceph1 ~]# ceph fs rm cephfs --yes-i-really-mean-it
[root@ceph1 ~]# ceph -s
  cluster:
    id:     87e6249e-8892-4f14-a73e-923952230ccf
    health: HEALTH_WARN
            no active mgr
 
  services:
    mon: 3 daemons, quorum ceph1,ceph2,ceph3
    mgr: no daemons active
    osd: 3 osds: 3 up, 3 in
 
  data:
    pools:   3 pools, 208 pgs
    objects: 11 objects, 665B
    usage:   3.02GiB used, 57.0GiB / 60.0GiB avail
    pgs:     208 active+clean
 [root@ceph1 ceph]# ceph osd pool rm cephfs_data cephfs_data  --yes-i-really-really-mean-it
 [root@ceph1 ceph]# ceph osd pool rm cephfs_metadata cephfs_metadata --yes-i-really-really-mean-it
pool 'cephfs_metadata' removed
[root@ceph1 ~]# ceph mds rm 0 
[root@ceph1 ceph]# ceph osd pool create cephfs_data 64
pool 'cephfs_data' created
[root@ceph1 ceph]# ceph osd pool create cephfs_metadata 64
pool 'cephfs_metadata' created
[root@ceph1 ceph]# ceph fs new cephfs cephfs_metadata cephfs_data
new fs with metadata pool 6 and data pool 5
[root@ceph1 ceph]# ceph mds stat
cephfs-1/1/1 up  {0=ceph1=up:active}
[root@ceph1 ceph]# ceph osd lspools
4 rbd,5 cephfs_data,6 cephfs_metadata,
[root@ceph1 ceph]# 

[root@ceph1 ~]# ceph mds fail 0
failed mds gid 24612

```
### cephFS

```
[root@ceph1 ~]# yum install ceph-fuse
[root@ceph1 ~]# mkdir myfs
[root@ceph1 ~]# ceph-fuse -m 10.10.10.1:6789 myfs
[root@ceph1 ~]# ceph-fuse -m 10.10.10.1:6789 myfs
ceph-fuse[7767]: starting ceph client
2019-12-05 10:31:17.292747 7f578348d0c0 -1 init, newargv = 0x7f578df667e0 newargc=9
ceph-fuse[7767]: starting fuse
[root@ceph1 ~]# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cl-root   50G  2.2G   48G   5% /
devtmpfs             478M     0  478M   0% /dev
tmpfs                489M     0  489M   0% /dev/shm
tmpfs                489M  6.7M  482M   2% /run
tmpfs                489M     0  489M   0% /sys/fs/cgroup
/dev/sda1           1014M  139M  876M  14% /boot
/dev/mapper/cl-home   47G   33M   47G   1% /home
tmpfs                489M   24K  489M   1% /var/lib/ceph/osd/ceph-0
tmpfs                 98M     0   98M   0% /run/user/0
ceph-fuse             60G  3.1G   57G   6% /root/myfs

[root@ceph1 ~]# umount myfs
[root@ceph1 ceph]# ceph-authtool --print-key /etc/ceph/ceph.client.admin.keyring 
AQA/AuFdPeamJhAAShWeFaR+CX8YDTuIOxjzGA==
[root@ceph1 ~]# mount -t ceph 10.10.10.1:6789:/ /root/myfs -o name=admin,secret=AQA/AuFdPeamJhAAShWeFaR+CX8YDTuIOxjzGA==
[root@ceph1 ~]# df -T
Filesystem          Type     1K-blocks    Used Available Use% Mounted on
/dev/mapper/cl-root xfs       52403200 2265584  50137616   5% /
devtmpfs            devtmpfs    488980       0    488980   0% /dev
tmpfs               tmpfs       499968       0    499968   0% /dev/shm
tmpfs               tmpfs       499968    6856    493112   2% /run
tmpfs               tmpfs       499968       0    499968   0% /sys/fs/cgroup
/dev/sda1           xfs        1038336  141676    896660  14% /boot
/dev/mapper/cl-home xfs       49250820   32944  49217876   1% /home
tmpfs               tmpfs       499968      24    499944   1% /var/lib/ceph/osd/ceph-0
tmpfs               tmpfs        99996       0     99996   0% /run/user/0
10.10.10.1:6789:/   ceph      62902272 3170304  59731968   6% /root/myfs
```

### daemon操作

```
[root@ceph1 ~]# ceph daemon mds.ceph1 help
{
    "cache drop": "drop cache",
    "cache status": "show cache status",
    "config diff": "dump diff of current config and default config",
    "config diff get": "dump diff get <field>: dump diff of current and default config setting <field>",
    "config get": "config get <field>: get the config value",
    "config help": "get config setting schema and descriptions",
    "config set": "config set <field> <val> [<val> ...]: set a config variable",
    "config show": "dump current config settings",
    "dirfrag ls": "List fragments in directory",
    "dirfrag merge": "De-fragment directory by path",
    "dirfrag split": "Fragment directory by path",
    "dump cache": "dump metadata cache (optionally to a file)",
    "dump loads": "dump metadata loads",
    "dump tree": "dump metadata cache for subtree",
    "dump_blocked_ops": "show the blocked ops currently in flight",
    "dump_historic_ops": "show slowest recent ops",
    "dump_historic_ops_by_duration": "show slowest recent ops, sorted by op duration",
    "dump_mempools": "get mempool stats",
    "dump_ops_in_flight": "show the ops currently in flight",
    "export dir": "migrate a subtree to named MDS",
    "flush journal": "Flush the journal to the backing store",
    "flush_path": "flush an inode (and its dirfrags)",
    "force_readonly": "Force MDS to read-only mode",
    "get subtrees": "Return the subtree map",
    "get_command_descriptions": "list available commands",
    "git_version": "get git sha1",
    "help": "list available commands",
    "log dump": "dump recent log entries to log file",
    "log flush": "flush log entries to log file",
    "log reopen": "reopen log file",
    "objecter_requests": "show in-progress osd requests",
    "ops": "show the ops currently in flight",
    "osdmap barrier": "Wait until the MDS has this OSD map epoch",
    "perf dump": "dump perfcounters value",
    "perf histogram dump": "dump perf histogram values",
    "perf histogram schema": "dump perf histogram schema",
    "perf reset": "perf reset <name>: perf reset all or one perfcounter name",
    "perf schema": "dump perfcounters schema",
    "scrub_path": "scrub an inode and output results",
    "session evict": "Evict a CephFS client",
    "session ls": "Enumerate connected CephFS clients",
    "status": "high-level status of MDS",
    "tag path": "Apply scrub tag recursively",
    "version": "get ceph version"
}
[root@ceph1 ~]# ceph daemon mds.ceph2 cache status 
[root@ceph1 ~]# ceph daemon mds.ceph1 perf dump mds
```

### ceph卸载

```
[root@ceph1 ~]# ceph-deploy purge ceph3
[root@ceph1 ~]# ceph-deploy purgedata ceph3
[root@ceph1 ~]# ceph-deploy forgetkeys
```

### osd 删除
```
[root@ceph1 ceph]# ceph osd out 0
marked out osd.0. 
[root@ceph1 ceph]# systemctl stop ceph-osd@0
[root@ceph1 ceph]# ceph osd crush remove osd.0
removed item id 0 name 'osd.0' from crush map
[root@ceph1 ceph]# ceph auth del osd.0
updated
[root@ceph1 ceph]# ceph osd rm 0
removed osd.0
[root@ceph1 ceph]# ceph-disk zap /dev/sdc
Creating new GPT entries.
GPT data structures destroyed! You may now partition the disk using fdisk or
other utilities.
Creating new GPT entries.
The operation has completed successfully.

ceph-disk 已经被ceph-volume替代，请使用ceph-volume
```