+++
title = '开源备份软件之Bup存储'
date = 2024-07-26T17:02:17+08:00
draft = false
tags = ['备份', '开源', 'Bup']
+++

## 存储

bup将其数据存储在git格式的存储库中。基本上，“bup split”读取数据，使用滚动校验和（类似于rsync）将其分解为块，并将这些块保存到一个新的git packfile中。

每个备份至少有一个git packfile。

在决定是否将特定块写入新的packfile时，bup首先检查存在的所有其他packfile，看看它们是否已经拥有该块。如果已经拥有，则会跳过该块。

git包分为两部分：包本身（.pack）和索引（.idx）。索引相当小，包含包中所有对象的列表。因此，在生成远程备份时，我们不必从远程服务器下载packfiles的全部副本：本地端只需下载服务器索引文件的副本，并在生成新包时将对象与对象进行比较，然后直接发送到服务器。

“bup split”和“bup save”的“-n”选项是您想要创建的备份的名称，但它实际上是作为git分支实现的。因此，您可以使用git检查特定分支，并接收大量与您拆分的文件相对应的分块文件。

如果您使用“-b”或“-t”或“-c”而不是“-n”，bup split将输出blob列表、包含该blob列表的树或分别包含该树的提交到stdout。您可以使用它来构建自己的脚本，用这些值做一些事情。

## index

“bup index”遍历文件系统，并更新一个文件（默认情况下，其名称为~/.bup/bupindex），以包含每个文件和目录的名称、属性和可选的git SHA1（blob id）。

“bup save”基本上只是运行相当于“bup split”的很多次，每个文件一次，并组装一个包含所有结果对象的git树。这使得“git diff”更有用。但是由于bup将大文件拆分为更小的块，因此生成的树结构与git本身存储的内容并不完全对应。

如果文件之前是由“bup save”写入的，则其git blob/tree id存储在索引中。这让“bup save”避免读取未更改的文件以产生大量增量备份。

## 加密压缩重删

1. 使用zlib库压缩压缩数据，压缩级别0-9
2. 加密暂不支持
3. 重删使用的C语言编写的hashsplit算法
