+++
title = '开源备份软件之BorgBackup 存储格式'
date = 2024-07-10T14:19:40+08:00
draft = false
tags = ['备份', '开源', 'BorgBackup']
+++

Borg 的Repository 是一个基于事务kv存储的文件系统，每个repository都有以下文件结构：

- README
  
  简单的文本文件，说明是Borg repository

- config

  repository 配置

- data/

  存储实际数据的目录

- hints.%d

  repository 压缩的hints

- index.%d

  repository 索引

- integrity.%d

  repository 完整性文件

不同与restic 的data，borg的data通过使用日志记录变化来实现的备份。日志是一系列编号的文件，称为segment. 每个segment由一系列日志条目构成。

## config

每个repository都有一个配置文件，这是一个INI风格的文件，如下所示：

```shell
    [repository]
    version = 1
    segments_per_dir = 1000
    max_segment_size = 524288000
    id = 57d6c1d52ce76a836b532b0e42e677dec6af9fca3673db511279358828a21ed6
```

其中repository.id是repository的唯一标识符。

## index

msgpack文件，使用事务id命名为``index.<TRANSACTION_ID>`` ，存储segments索引信息。

## hints

msgpack文件，使用事务id命名为``hints.<TRANSACTION_ID>`` ，包括：

```python
        hints = {
            b'version': 2,
            b'segments': self.segments,
            b'compact': self.compact,
            b'storage_quota_use': self.storage_quota_use,
            b'shadow_index': self.shadow_index,
        }
```

主要是 segments 和compact信息。

## integrity

msgpack文件，使用事务id命名为``integrity.<TRANSACTION_ID>`` ，包括：

```python
        integrity = {
            b'version': 2,
            b'index': index.integrity_data,
            b'hints': hints.integrity_data,
        }
```

主要是 index 和hints 的校验信息。

## data

存储segment日志信息文件。

一个segment以一个魔术数字（BORG_SEG作为八字节的ASCII字符串）开头，然后是多个日志条目。每个日志条目包括：（按此顺序）

- 无符号32位数字，整个条目的CRC32（不包括CRC32字段）

- 无符号32位的条目大小（包括整个header）

- 无符号的8位条目标签：PUT（0）、DELETE（1）或COMMIT（2）

- PUT或DELETE，32字节Key

- PUT，（size-41）字节的数据（len=size-sizeof(CRC32)-sizeof(size)-sizeof(tag)-sizeof(key)）

当repository事务提交时，会写入COMMIT标签。包含COMMIT的段的段号就是TRANSACTION_ID。

当打开repository时，任何不后跟COMMIT标签的PUT或DELETE操作都会被丢弃，因为它们是未提交事务的一部分。

![segment](segment.png)
