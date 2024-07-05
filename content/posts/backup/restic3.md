+++
title = '开源备份软件之Restic加密压缩重删'
date = 2024-07-05T10:09:41+08:00
draft = false
tags = ['备份', '开源', 'restic']
+++

前面提到了restic的目录布局逻辑，总结起来就是：

- snapshots直接指向了最顶层的tree blob
- index目录下的文件记录了每一次备份snapshot的完整组织结构，指向packs以及包含的blobs
- data目录中是packs id，每个packs中可能包含多个blob
- blob包含tree和data两种类型，pack也分这两种类型，tree blob记录了与文件有关的metadata信息，content到具体data blob

## 重删

restic 的重删就是依据data blob, blob只会在repository中存储一次。

每个文件中的数据被拆分为可变长度的Blobs，以64字节的滑动窗口定义的偏移量切割，使用Rabin Fingerprints来实现Content Defined Chunking (CDC)。

小于512 KiB的文件不拆分，Blob的大小为512 KiB至8 MiB。实施的目标是平均1 MiB Blob大小。

相关部分代码实现独立与restic项目 [chunker](https://github.com/restic/chunker)。

## 压缩

repository V2 版本支持压缩，数据压缩算法使用的ZSTD。

pack压缩的对象还是bolb，blob中包含未压缩数据的长度。首先会计算blob的 sha256 哈希，如果是新的数据则压缩后再上传。

index、lock、snapshot 等文件加密前会被压缩。

目前还不支持数据repository下压缩级别调整，兼容性也有问题，需要手动处理。

## 加密

restic存储在repository中的所有数据都在CTR模式下使用AES-256加密，并使用Poly1305-AES进行身份验证。

为了加密新数据，前16个字节从加密安全的伪随机数生成器中读取为随机nonce。

此操作需要三个密钥：用于加密的AES-256的32字节，用于加密的16字节的AES密钥和用于Poly1305的16字节密钥。

用AES-256加密数据，通过密文计算消息身份验证代码（MAC），然后将所有内容存储为IV || CIPHERTEXT || MAC。

目录key包含了密钥文件。
