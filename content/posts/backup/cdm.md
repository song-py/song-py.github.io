+++
title = 'CDM'
date = 2024-05-30T22:53:43+08:00
tags = ['备份','CDM']
draft = true
+++

CDM(copy data manager) 随着 Cohesity、Rubrik和Actifio 风头正劲，提出了原格式，黄金副本等管理技术。

CDM一般采用永久增量的备份方式，copy原格式数据，并可以直接将copy的原格式数据挂载进行即时恢复，容灾，开发，测试，合规性审计，分析等操作。

相对于之前的cdp技术，都是原格式，都可以利用cbt 的方式实现，CDM更注重副本集的管理。

CDM的快照加上数据库的日志可以回滚到任意时间点。

对于虚拟机和云主机，一般是利用原生的快照接口处理，数据库则是入侵主机的cbt的驱动，如果不想使用驱动，需要物理备份原格式文件。
