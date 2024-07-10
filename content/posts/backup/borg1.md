+++
title = '开源备份软件之BorgBackup简介'
date = 2024-07-08T14:26:02+08:00
draft = false
tags = ['备份', '开源', 'BorgBackup']
+++


[BorgBackup](https://github.com/borgbackup/borg) 是一个 Python 和 C 语言编写的命令行增量数据备份工具

## 主要优势（总结自官方文档）

- 高效：BorgBackup 会将文件按数据块去重，只有改动的数据块才会被备份。一个 25 GiB 的虚拟机磁盘文件，只改动了 1 GiB，那就只会新增备份这 1 GiB 的数据；
- 高速：核心算法使用 C 编译，使用缓存快速跳过未改动过的文件以加快备份速度；
- 加密：数据默认是 AES-256 加密并且 HMAC-SHA256 校验的；
- 压缩：支持多种压缩算法，可自动检测数据是否属于可被压缩的类型；
  - LZ4 快，低压缩
  - ZSTD 高速低压缩、低速高压缩
  - ZLIB 中等速度，中等压缩
  - LZMA 低速 高压缩
- 异地备份：原生支持 SSH 备份到异地服务器，也可使用 NFS 等网络存储；
- 可挂载：可以直接用 FUSE 挂载一个备份存档读取里面的数据；
- 跨平台：支持 Linux, macOS, BSD, Windows (Cygwin / WSL) 等多种平台；
- 开源：安全可审计，易于修改。

## 客户端

BorgBackup 由 Python + C 写成。可以直接从源代码编译，从 PyPI 安装，或者使用官方 PyInstaller 制作的单文件程序。

目前Borg 最新发行版本是1.4版本，同时有2.0开发版本，但是不建议使用，并且2版本不兼容1版本。

Borg 本身是一个命令行工具，但是macOS 和 Linux 下还可以使用[vorta](https://github.com/borgbase/vorta)，GUI客户端。

## 与restic的区别

- restic有云后端，Borg通过ssh到远程Borg做客户端/服务器
- Borg支持多种压缩算法，rstic最新版本只有zstd
- Borg备份存储库仅由一个系统使用, restic 可以多个系统使用，重删效率更高
- Borg没有多线程
- Borg的windows版本支持比较麻烦
