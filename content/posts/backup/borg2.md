+++
title = '开源备份软件之BorgBackup 使用'
date = 2024-07-09T14:40:23+08:00
draft = true
+++

与restic相同， BorgBackup 中也有两个基本概念：

- repository 备份仓库
- archive 每次备份

## 基本命令

``` shell
borg init -- 初始化一个repository
borg create -- 创建一个archive到repository中
borg list -- 列出所有的repository或者某个repository中某个archive的内容
borg extract -- 还原某个archive
borg delete -- 手动删除某个archive
borg config -- 获取或者设置某个配置
```

## 流程

创建仓库存储目录并初始化：

```shell
$ mkdir -p /tmp/backup/
#初始化 repository 的时候，可以指定加密类型
$ borg init --encryption=repokey /tmp/backup/borg_sample
```

第一次备份:

```shell
$ borg create --stats --progress /tmp/backup/borg_sample::first /tmp/data/
------------------------------------------------------------------------------
Archive name: first
Archive fingerprint: bd5ddf9ac944353bb209576c12a16652a18f866c9d033fef2eb582d288a3cff5
Time (start): Sat, 2022-01-29 10:48:07
Time (end):   Sat, 2022-01-29 10:48:07
Duration: 0.54 seconds
Number of files: 3
Utilization of max. archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:              104.86 MB            105.27 MB            105.27 MB
All archives:              104.86 MB            105.27 MB            105.27 MB

                       Unique chunks         Total chunks
Chunk index:                      42                   42
------------------------------------------------------------------------------
```

如果不传 --stats --progress 则静默输出。

在 create 的时候可以选择使用压缩算法，如果不指定压缩算法，默认会使用 LZ4：

```shell
borg create --compress zstd,1 /tmp/backup/borg_sample::zstd /tmp/data/
borg create --compress none /tmp/backup/borg_sample::none /tmp/data/
```

列出备份:

```shell
$ borg list /tmp/backup/borg_sample/
first                                Sat, 2022-01-29 10:48:07 [bd5ddf9ac944353bb209576c12a16652a18f866c9d033fef2eb582d288a3cff5]
second                               Sat, 2022-01-29 10:49:27 [6a5131bc1098d4ba46daabc6e440d6daf9211db94ac2d17893f996ce78840162]


$ borg list /tmp/backup/borg_sample::first
drwxrwxr-x einverne einverne        0 Sat, 2022-01-29 10:47:39 tmp/data
-rw-rw-r-- einverne einverne 104857600 Sat, 2022-01-29 10:47:41 tmp/data/ramdom.dump
-rw-rw-r-- einverne einverne       11 Sat, 2022-01-29 10:47:11 tmp/data/file_change.txt
-rw-rw-r-- einverne einverne       12 Sat, 2022-01-29 10:47:21 tmp/data/file_static.txt
```

恢复备份：

```shell
$ borg extract --list /tmp/backup/borg_sample::first
tmp/data
tmp/data/file_change.txt
tmp/data/file_static.txt
tmp/data/random.dump

$ borg extract --list /tmp/backup/borg_sample::second
tmp/data
tmp/data/file_change.txt
tmp/data/file_static.txt
tmp/data/random.dump
tmp/data/file_new.txt
tmp/data/random_2.dump
```

删除备份：

```shell
$ borg delete /tmp/backup/borg_sample::first
$ borg list /tmp/backup/borg_sample
second                               Tue, 2019-09-24 04:10:55 [a423a94e8a8f4352e72c0951e6a408f4f4f6d5f362518dcbcba77b9005dafa12]
```

## 其他命令

此外还有一些常用命令

```shell
borg diff /tmp/backup/borg_sample::first second   -- 查找archive之间的差异
borg rename /tmp/backup/borg_sample::first begin  -- 重命名archive
borg mount /tmp/backup/borg_sample::first /tmp/mymountpoint  -- fuse挂载archive
borg umount /tmp/mymountpoint  -- fuse卸载archive
borg prune -v --list --keep-daily=7 --keep-weekly=4 --keep-monthly=-1 /tmp/backup/borg_sample  -- 配置archive保留规则
borg compact /tmp/backup/borg_sample -- 压缩repository 释放空间
borg info /tmp/backup/borg_sample  -- 查询repository或者archive信息
```
