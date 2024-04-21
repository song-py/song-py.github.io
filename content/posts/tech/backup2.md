+++
title = '总体概念'
date = 2024-04-21T20:49:54+08:00
tags = ['备份','容灾']
draft = false
+++

先介绍些基本概念：

1. 复制（replication）： 就是拷贝数据到另外一个地方。
2. 备份（backup）； 就是将数据复制一份到另外一个地方，可以说是一种特殊的复制。 如果数据不小心误删除了，拷贝回来就是恢复(restore)；
3. 容灾（Disaster Tolerance）：就是发生各种灾害的时候，系统能够正常运行；灾难发生后，将系统恢复到正常运行状态就是 灾难恢复（Disaster Recovery）；

备份是所有操作的基石，系统的恢复依赖于备份，各种主从、双活、两地三中心的容灾基础还是备份。

容灾有两个关键的参数：

1. RPO（Recovery Point Object）：这个是系统恢复后丢失了多长时间的数据；
2. RTO（Recovery Time Object）：这个是多长时间能恢复系统；

一般来说，各家吹嘘的比较厉害，但是一崩溃就最少几个小时。
另外还有些新的指标：

1. RRO（Recovery Reliability Objective）：这个是系统能成功恢复的比例；
2. RIO（Recovery Integrity Objective）： 这个是系统恢复后数据的完整性比例；
3. DOO（Degraded Operations Objective）： 这个是允许系统部分恢复到什么程度；
4. NRO（NetWork Recovery Objective）： 这个是切换到灾备中心的时间；
