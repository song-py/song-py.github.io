+++
title = '开源备份软件之Restic存储格式'
date = 2024-07-04T19:06:17+08:00
draft = true
tags = ['备份', '开源', 'restic']
+++

restic的存储术语为repository，备份期间生成的所有数据都以结构化形式发送到repository并存储在repository中。

## Repository 布局

最基本的布局如下所示：

    /tmp/restic-repo
    ├── config
    ├── data
    │   ├── 21
    │   │   └── 2159dd48f8a24f33c307b750592773f8b71ff8d11452132a7b2e2a6a01611be1
    │   ├── 32
    │   │   └── 32ea976bc30771cebad8285cd99120ac8786f9ffd42141d452458089985043a5
    │   ├── 59
    │   │   └── 59fe4bcde59bd6222eba87795e35a90d82cd2f138a27b6835032b7b58173a426
    │   ├── 73
    │   │   └── 73d04e6125cf3c28a299cc2f3cca3b78ceac396e4fcf9575e34536b26782413c
    │   [...]
    ├── index
    │   ├── c38f5fb68307c6a3e3aa945d556e325dc38f5fb68307c6a3e3aa945d556e325d
    │   └── ca171b1b7394d90d330b265d90f506f9984043b342525f019788f97e745c71fd
    ├── keys
    │   └── b02de829beeb3c01a63e6b25cbd421a98fef144f03b9a02e46eff9e2ca3f0bd7
    ├── locks
    ├── snapshots
    │   └── 22a5af1bdc6e616f8a29579458c49627e01b32210d09adb288d1ecda7c5711ec

repository由几个目录和一个称为config的顶级文件组成。对于存储在repository中的所有其他文件，文件的名称是存储ID的小写十六进制表示，这是文件内容的SHA-256散列。这允许通过在文件上运行程序sha256sum并将其输出与文件名进行比较，即可轻松验证文件的意外修改。
除了存储在keys和data目录中的文件外，所有文件都在计数器模式（CTR）下使用AES-256加密。加密数据的完整性由Poly1305-AES消息身份验证代码（MAC）保护。data目录中的文件（“pack”）由多个部分组成，这些部分都经过独立加密和身份验证

## Config

config文件用加密文件保存下面信息。

```Golang
// Config contains the configuration for a repository.
type Config struct {
 Version           uint        `json:"version"`
 ID                string      `json:"id"`
 ChunkerPolynomial chunker.Pol `json:"chunker_polynomial"`
}
```

在加密文件的前16个字节中，存储初始化矢量（IV）。它后面是加密的数据，并最后填充16字节的MAC。格式为：IV||CIPHERTEXT||MAC。完整的加密开销为32字节。

其中，id是repository的唯一标识，chunker_polynomial 用于切分大文件。

## Data

data 目录存放的pack文件是有一个或者多个blob组成，pack的数据格式如下:
    EncryptedBlob1 || ... || EncryptedBlobN || EncryptedHeader || Header_Length

```golang
// Packer is used to create a new Pack.
type Packer struct {
 blobs []restic.Blob

 bytes uint
 k     *crypto.Key
 wr    io.Writer

 m sync.Mutex
}
```

Pack文件的末尾是一个header，它描述了内容。header经过加密和身份验证。Header_Length是加密标头的长度，在小端编码中编码为四字节整数。将header放在文件末尾，可以在备份阶段读取blob后立即将其写入连续流中。这降低了代码的复杂性，并避免了一旦包完成并知道header的内容和长度后必须重写文件。
所有blob都独立进行身份验证和加密，解密后，Pack由以下元素组成：
    Type_Blob1 || Data_Blob1 ||
    [...]
    Type_BlobN || Data_BlobN ||

Blob type 字段是一个字节，具体定义:

```
+-----------+----------------------+-------------------------------------------------------------------------------+
| Type      | Meaning              |  Data                                                                         |
+===========+======================+===============================================================================+
| 0b00      | data blob            |  ``Length(encrypted_blob) || Hash(plaintext_blob)``                           |
+-----------+----------------------+-------------------------------------------------------------------------------+
| 0b01      | tree blob            |  ``Length(encrypted_blob) || Hash(plaintext_blob)``                           |
+-----------+----------------------+-------------------------------------------------------------------------------+
| 0b10      | compressed data blob |  ``Length(encrypted_blob) || Length(plaintext_blob) || Hash(plaintext_blob)`` |
+-----------+----------------------+-------------------------------------------------------------------------------+
| 0b11      | compressed tree blob |  ``Length(encrypted_blob) || Length(plaintext_blob) || Hash(plaintext_blob)`` |
+-----------+----------------------+-------------------------------------------------------------------------------+
```

这足以计算Pack中所有Blob的偏移量。在数据列中， ``Length(plaintext_blob)`` 是指blob包含的解密和未压缩数据的长度。
Blob可以使用zstd算法压缩。
data和tree必须存储在单独的文件中。相同类型的压缩和非压缩blob可以混合在打包文件中。
tree blob 记录了备份文件和目录的元数据信息以及目录结构。

```golang
// Node is a file, directory or other item in a backup.
type Node struct {
 Name       string      `json:"name"`
 Type       string      `json:"type"`
 Mode       os.FileMode `json:"mode,omitempty"`
 ModTime    time.Time   `json:"mtime,omitempty"`
 AccessTime time.Time   `json:"atime,omitempty"`
 ChangeTime time.Time   `json:"ctime,omitempty"`
 UID        uint32      `json:"uid"`
 GID        uint32      `json:"gid"`
 User       string      `json:"user,omitempty"`
 Group      string      `json:"group,omitempty"`
 Inode      uint64      `json:"inode,omitempty"`
 DeviceID   uint64      `json:"device_id,omitempty"` // device id of the file, stat.st_dev, only stored for hardlinks
 Size       uint64      `json:"size,omitempty"`
 Links      uint64      `json:"links,omitempty"`
 LinkTarget string      `json:"linktarget,omitempty"`
 // implicitly base64-encoded field. Only used while encoding, `linktarget_raw` will overwrite LinkTarget if present.
 // This allows storing arbitrary byte-sequences, which are possible as symlink targets on unix systems,
 // as LinkTarget without breaking backwards-compatibility.
 // Must only be set of the linktarget cannot be encoded as valid utf8.
 LinkTargetRaw      []byte                                   `json:"linktarget_raw,omitempty"`
 ExtendedAttributes []ExtendedAttribute                      `json:"extended_attributes,omitempty"`
 GenericAttributes  map[GenericAttributeType]json.RawMessage `json:"generic_attributes,omitempty"`
 Device             uint64                                   `json:"device,omitempty"` // in case of Type == "dev", stat.st_rdev
 Content            IDs                                      `json:"content"`
 Subtree            *ID                                      `json:"subtree,omitempty"`

 Error string `json:"error,omitempty"`

 Path string `json:"-"`
}
```

其中subtree 链接转到子目录tree blob。

## Index

Index 内文件包含Blob以及它们所包含的Pack的信息。当本地缓存索引无法再访问时，可以下载索引文件并用于重建索引。

```golang
// Index holds lookup tables for id -> pack.
type Index struct {
 m      sync.RWMutex
 byType [restic.NumBlobTypes]indexMap
 packs  restic.IDs

 final   bool       // set to true for all indexes read from the backend ("finalized")
 ids     restic.IDs // set to the IDs of the contained finalized indexes
 created time.Time
}
```

## Keys

目录keys包含密钥文件,从用户密码中导出存储库主加密和消息身份验证密钥所需的所有数据。

```golang
type Key struct {
 Created  time.Time `json:"created"`
 Username string    `json:"username"`
 Hostname string    `json:"hostname"`

 KDF  string `json:"kdf"`
 N    int    `json:"N"`
 R    int    `json:"r"`
 P    int    `json:"p"`
 Salt []byte `json:"salt"`
 Data []byte `json:"data"`

 user   *crypto.Key
 master *crypto.Key

 id restic.ID
}
```

## Snapshot

快照表示在给定时间点包含所有文件和子目录的目录。对于每个备份，都会创建一个新的快照存储在repository中目录snapshots下方的文件中。文件名是快照id。

snapshot 主要包括下面内容：

```golang
// Snapshot is the state of a resource at one point in time.
type Snapshot struct {
 Time     time.Time `json:"time"`
 Parent   *ID       `json:"parent,omitempty"`
 Tree     *ID       `json:"tree"`
 Paths    []string  `json:"paths"`
 Hostname string    `json:"hostname,omitempty"`
 Username string    `json:"username,omitempty"`
 UID      uint32    `json:"uid,omitempty"`
 GID      uint32    `json:"gid,omitempty"`
 Excludes []string  `json:"excludes,omitempty"`
 Tags     []string  `json:"tags,omitempty"`
 Original *ID       `json:"original,omitempty"`

 ProgramVersion string           `json:"program_version,omitempty"`
 Summary        *SnapshotSummary `json:"summary,omitempty"`

 id *ID // plaintext ID, used during restore
}
```

parent 是上次snapshot 快照id， tree是关键ID可以定位到对应blob。

## Locks

repository设计允许多个restic并行访问甚至并行写入实例。然而，有一些函数需要对repository进行独家访问。为了实现这些功能，restic需要在做任何事情之前在repository上创建一个锁。

锁有两种类型：独家锁和非独家锁。最多一个进程可以在repository上有一个排他性锁，在此期间，不得有任何其他锁（排他性和非排他性）。可能同时存在多个非排他性锁。

当要创建新锁时，restic会检查repository中的所有锁。当找到锁时，会测试锁是否过时。如果锁是在同一台机器上创建的，即使是年轻的锁，也会通过向其发送信号来测试该过程是否仍然有效。如果失败，restic假设该过程已死，并认为锁已过时。

当要创建新锁且未检测到其他冲突锁时，restic会创建一个新锁，等待并检查存储库中是否出现其他锁。根据其他锁的类型和要创建的锁，restic要么继续，要么失败。

```golang
repositorytype lockContext struct {
 lock      *restic.Lock
 cancel    context.CancelFunc
 refreshWG sync.WaitGroup
}

type locker struct {
 retrySleepStart       time.Duration
 retrySleepMax         time.Duration
 refreshInterval       time.Duration
 refreshabilityTimeout time.Duration
}
```
