+++
title = 'Bool未初始化的奇怪问题'
date = 2024-03-13T21:11:14+08:00
tags = ['C++']
draft = false
+++

一个bool未初始化的问题居然搞了几个小时， 赋值语句类似下面。

```C++
   flag = attr.is_link ? 1 : 0;
```

其中  is_link 在其他函数处理的时候未初始化，导致flag出现了一个奇怪数值，即不是1，也不是0。

bool未初始化的随机值居然影响到后续的赋值。

看来还是要用lint工具检查了。
