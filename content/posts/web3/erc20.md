+++
title = 'ERC20标准'
date = 2024-07-23T10:48:48+08:00
draft = true
tags = ['web3', '以太坊']
+++

ERC20（Ethereum Request for Comments 20）是以太坊上的一种代币标准，它定义了一组接口（方法和事件），使得代币可以在不同的应用程序、钱包和交易所之间进行互操作。

[EIP-20](https://eips.ethereum.org/EIPS/eip-20) 中提出。ERC20 标准使得创建和使用代币变得简单和一致，是最广泛采用的代币标准之一。

## EIP

EIP（Ethereum Improvement Proposal，以太坊改进提案）是以太坊社区提出和讨论对以太坊平台进行改进和变更的正式流程。EIP 分为几个主要分类，每个分类有不同的目的和用途。以下是 EIP 的主要分类：

- 核心（Core）：

这些是影响以太坊核心协议的改进提案，包括共识协议更改、网络升级和硬分叉。
Core EIP 是对以太坊的底层协议和运行机制的改进，需要进行广泛的测试和社区共识。

- 接口（Interface，ERC）：

这些提案用于标准化以太坊上的应用程序接口（API）和智能合约标准。
最著名的例子是 ERC-20 标准，它定义了代币的接口，使代币在不同的应用程序和交易所之间具有互操作性。

- 网络（Networking）：

涉及以太坊网络协议的改进提案，包括对节点间通信的更改。
Networking EIP 包括对以太坊网络消息传递协议和数据传输的改进。

- 元（Meta）：

Meta EIP 涉及以太坊改进提案流程本身的改进，包括对 EIP 模板、格式和流程的更改。
这些提案不直接涉及以太坊代码或协议的更改，而是关于管理和协调改进提案的流程。

- 信息（Informational）：

提供与以太坊设计和使用相关的指导和信息。
这些提案不提议任何新的功能或标准，只是为开发者和用户提供信息和最佳实践。

### 接口

ERC20 标准提供了一组基础接口，使得代币可以在以太坊生态系统中方便地创建、管理和交换。

```solidity
pragma solidity ^0.8.20;

interface IERC20 {
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    function totalSupply() external view returns (uint256);
    function balanceOf(address account) external view returns (uint256);
    function transfer(address to, uint256 value) external returns (bool);
    function allowance(address owner, address spender) external view returns (uint256);
    function approve(address spender, uint256 value) external returns (bool);
    function transferFrom(address from, address to, uint256 value) external returns (bool);
}
```

### 代币

区分原生币（Coin）和代币（Token）的关键概念：

- 原生币（Coin）

原生币: 以太币（Ether，ETH）是以太坊区块链的原生加密货币。它直接由区块链协议生成和管理。
用途: 主要用于支付网络上的交易费用（Gas）以及奖励矿工（现在是验证者）。
特性: 原生币的交易是直接在区块链上进行的，不需要任何智能合约。
地址: 以太币的交易地址是由以太坊协议生成的。

- ERC20 代币（Token）

智能合约币: ERC20 代币是通过智能合约创建和管理的加密货币。它们不是区块链的原生币，而是构建在区块链之上的。
用途: 可以代表各种资产或功能，如稳定币、权益证明、治理代币等。
特性: ERC20 代币遵循以太坊改进提案 20（EIP-20）的标准，实现了一组基本的接口和功能，使其能够在去中心化应用（DApps）之间互操作。
地址: ERC20 代币的合约地址是由智能合约生成的。

- 包装以太币（Wrapped Ether）

包装以太币: WETH（Wrapped Ether）是将 ETH 包装成 ERC20 代币的形式，使其能够在需要 ERC20 标准的去中心化应用中使用。
用途: 由于 ETH 不是 ERC20 代币，有些 DApps 需要使用 ERC20 标准，因此 WETH 充当桥梁，使 ETH 可以像其他 ERC20 代币一样使用。
特性: 1 WETH 始终等于 1 ETH，用户可以随时在 WETH 和 ETH 之间转换。
合约地址: WETH 有自己的智能合约地址，用户通过该地址进行 WETH 与 ETH 的转换。
