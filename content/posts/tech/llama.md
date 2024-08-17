+++
title = 'Llama.cpp'
date = 2024-08-17T21:44:25+08:00
draft = false
tags = ['AI','Llama']
+++

新公司要搞AI 增加产品竞争力，但是又没有GPU服务器，研究了一下Llama.cpp 应该可以满足要求。

Llama.cpp 由 Georgi Gerganov 开发。它以高效的 C/C++ 实现了 Meta 的 Llama 架构,以其通用兼容性、综合功能集成和专注的优化而脱颖而出.

Llama.cpp 的主干是原始的 Llama 模型，它也基于 Transformer 架构:

Llama 架构与 Transformer 架构的主要区别：

![架构图](llama.png)

- 预归一化（GPT3）：用于通过使用 RMSNorm 方法对每个 Transformer 子层的输入进行归一化来提高训练稳定性，而不是对输出进行归一化。

- SwigGLU激活函数（PaLM）：将原来的非线性ReLU激活函数替换为SwiGLU激活函数，从而带来性能提升。

- 旋转嵌入（GPTNeao）：在删除绝对位置嵌入后，在网络的每一层添加旋转位置嵌入（RoPE）。

开始使用 Llama.cpp 的先决条件包括：

1. 系统应有make（MacOS/Linux自带）或cmake（Windows需自行安装）编译工具
2. 建议使用Python 3.10以上编译和运行该工具

后续就可以从 GitHub 上克隆 Llama.cpp 的仓库，并按照官方文档进行安装运行。
