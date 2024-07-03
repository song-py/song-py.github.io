+++
title = '开源备份软件之Restic简介'
date = 2024-07-03T10:46:53+08:00
draft = false
tags = ['备份', '开源', 'restic']
+++

## 简介

[restic](https://github.com/restic/restic) 是一个简单快速安全的备份程序，支持文件的备份。

centos可以直接yum 安装，也支持docker容器，也可以通过源码编译，源码是go编写，需要安装go编译器。

存储池支持:

- Local directory
- sftp server (via SSH)
- HTTP REST server (protocol, rest-server)
- Amazon S3 (either from Amazon or using the Minio server)
- Alibaba Cloud (Aliyun)
- OpenStack Swift
- BackBlaze B2
- Microsoft Azure Blob Storage
- Google Cloud Storage
- rclone Backend

## 使用

备份前需要为备份集先创建并初始化repository，并且新建密码：

    $ restic init --repo /srv/restic-repo
    enter password for new repository:
    enter password again:
    created restic repository 085b3c76b9 at /srv/restic-repo
    Please note that knowledge of your password is required to access the repository.
    Losing your password means that your data is irrecoverably lost.

不同的backend repository需要不同的参数。

备份使用下面命令并且输入repository 密码， -v输出备份信息，-vv输出更详细文件信息。

    $ restic -r /srv/restic-repo -vv backup ~/work.txt
    open repository
    enter password for repository:
    lock repository
    load index files
    using parent snapshot f3f8d56b
    start scan
    start backup
    scan finished in 2.115s
    modified  /home/user/work.txt, saved in 0.007s (22 B added)
    modified  /home/user/, saved in 0.008s (0 B added, 378 B metadata)
    modified  /home/, saved in 0.009s (0 B added, 375 B metadata)
    processed 22 B in 0:02
    Files:           0 new,     1 changed,     0 unmodified
    Dirs:            0 new,     2 changed,     0 unmodified
    Data Blobs:      1 new
    Tree Blobs:      3 new
    Added:      1.116 KiB
    snapshot 8dc503fc saved

备份完毕输入snapshot id。

需要恢复文件时直接输入恢复命令：

    $ restic -r /srv/restic-repo restore 8dc503fc --target /tmp/restore-work
    enter password for repository:
    restoring <Snapshot of [/home/user/work] at 2015-05-08 21:40:19.884408621 +0200 CEST> to /tmp/restore-work

同时也支持挂载恢复：

    $ mkdir /mnt/restic
    $ restic -r /srv/restic-repo mount /mnt/restic
    enter password for repository:
    Now serving /srv/restic-repo at /mnt/restic
    Use another terminal or tool to browse the contents of this folder.
    When finished, quit with Ctrl-c here or umount the mountpoint.

## 设计

备份：

- 不支持定时策略备份，需要配置系统定时任务
- windows 支持 VSS 快照
- 存储仓库支持多个数据对象，通过host,paths 来区分
- 之前的备份快照被称为父快照，自动匹配最新的父快照比对增量，也支持指定
- unix 是通过：mtime、ctime、file size、inode number 来确认是否变化，也可以用参数关闭某个匹配原则
- 无备份数据默认创建快照，但是也支持不创建快照
- 支持dry run 模式
- 备份时支持排除文件和包括文件，列表可以通过文件指定
- 链接文件不穿透读取，挂载目录会读取到内部，一些特殊元数据也默认不会读取
- 支持备份命令的输出，并且在恢复时导出，可以脚本方式处理数据库
- 支持备份异常退出后到续传

快照：

- 支持ls 命令查看快照文件信息
- 支持find 命令查找快照内文件
- 支持diff 命令比较两个快照差异
- 支持copy 命令复制快照到另一个repository
- 支持check 命令检查快照数据是否正确，并repair 命令修复repository
- 支持forget 命令删除快照，该命令只是逻辑删除，需要物理清理，必须使用prune命令
- 支持根据策略删除多个快照
  
恢复：

- 恢复时支持指定排除文件和包含文件
- 恢复时默认覆盖，同时提供修改覆盖、更新覆盖、不覆盖等参数
- 支持使用FUSE在unix 提供挂载
