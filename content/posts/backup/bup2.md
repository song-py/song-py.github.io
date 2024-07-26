+++
title = '开源备份软件之Bup使用'
date = 2024-07-26T10:49:29+08:00
draft = false
tags = ['备份', '开源', 'Bup']
+++

Bup直接使用命令行处理，同时也有前端KDE Kup。

## 基本命令

``` shell
bup init -- 初始化一个repository
bup index -- 创建文件索引
bup save -- 创建备份集到repository中
bup restore -- 从备份集恢复数据
bup ls -- 列出respository中的内容
bup fuse -- 挂载repository备份作为文件系统
bup rm -- 逻辑删除备份achive
bup gc -- 物理回收数据
```

## 流程

初始化默认的bup存储库（默认~/.bup 目录）:

``` shell
bup init
```

进行本地备份（-v或-vv输入详细信息）:

``` shell
bup index /etc
bup save -n local-etc /etc
```

将本地备份恢复到./dest：

``` shell
bup restore -C ./dest local-etc/latest/etc
ls -l dest/etc
```

## 其他命令

此外还有一些常用命令，参见[man](https://bup.github.io/man.html)
