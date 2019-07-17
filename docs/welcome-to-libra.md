---
id: welcome-to-libra
title: 欢迎
---

**欢迎来到Libra开发者网站！** Libra致力于为数十亿人建立一个简明的全球性货币金融基础体系。

> 世界所迫切需要的是通过可靠的数字货币及其所依托的底层框架结合，实现“货币互联网”。在移动设备上保护金融资产的安全性应该更加简单。不管你在何地，是何职业，工资几何，国际间的资金流动都应像发短信或分享图片一样简单有效且安全可靠。--[Libra白皮书](https://libra.org/en-us/whitepaper)

Libra建立在一个可扩展的且安全可靠的区块链基础设施上。它的价值映射了现实中的资产储备，由致力于推动金融体系演变的独立组织Libra联盟监管。

> Libra区块链的目标是为数十亿人提供金融服务的基础设施，其中也包括新的全球货币的，。Libra区块链从建造之初，就以可扩展性，安全性，存储及吞吐的效率，和未来的可变性为优先考量。— [Libra白皮书](https://libra.org/en-us/whitepaper)

Libra货币以Libra区块链为基础。Libra Core项目是Libra协议下的开源项目，也是Libra项目的技术支持。测试网络[（testnet）](reference/glossary.md#testnet)是这个新系统的Demo。和即将推出的Libra[主网](reference/glossary.md#mainnet)不同，测试网络中使用的货币没有价值，可以凭空产生。

本文档包括：

* Hello World：通过测试网络实现第一笔[交易](my-first-transaction.md)；
* 学习：Libra协议，Move语言，LibraBFT等新技术；
* 社区：如何成为Libra生态的一部分。

<blockquote class="block_note">

**注意：** Libra项目仍处于原型阶段。Libra协议和Libra Core API并非正式版本。
原型项目的关键任务之一是完善协议和API。目前我们的重点在基础架构的构建和CLI客户端的设计上。路线图中，我们下一步的目标是公共API及其相关的库。我们欢迎在测试网络对软件进行相关测试，但是开发人员应了解，使用这些API发布应用程序仍然需要一些工作。我们也会发布我们的有关进展。
</blockquote>

## Move: 一种新的区块链编程语言

Move是一种新的编程语言，用于在Libra区块链上实现自定义交易逻辑和“智能合约”因为Libra的目标在于有朝一日为数十亿人服务，所以Move语言把安全性放在首位。

Move从过往的涉及智能合约的安全事故中汲取教训，创建了一种更易于表达和编写的语言。这将有效降低意外错误或安全事故的风险。具体而言，Move语言的设计可防止资产被复制。他可以通过定义资源类型使得数字资产和现实资产具有相同特性：单一所有者，仅能使用一次且资源的创建受限。

Move使关键交易代码的开发变得更加简单。它可以更为安全的实现Libra生态的治理准则，例如对Libra货币的管理以及对网络验证节点的管理。我们预计开发人员创建合约的能力将随着时间推移不断增强。这也将推动Move的演进和落实。

参考：[进一步了解Move语言](move-overview.md)


## Libra生态系统

Libra生态系统包括以下不同类型的实体：

* [客户端](#clients)
* [Validator 节点](#validator-nodes)
* [开发者](#developers)

### 客户端

Libra客户端:

* 是一种可以于Libra区块链交互的软件 
* 可由终端用户或代表终端用户的系统运行（如托管客户端） 
* 允许用户创建，签名和并提交交易给 [validator node](reference/glossary.md#validator-node).
* 能够向Libra区块链（通过[consensus protocol](reference/glossary.md#consensus-protocol) ）发送请求，请求交易或者账户状态，并且验证响应。

Libra Core包含可向测试网络提供交易请求的客户端。

体验：通过测试网络实现第一笔交易。将指导大家使用Libra CLI客户端进行[第一笔交易](my-first-transaction.md) 。

### Validator 节点  

[Validator 节点](reference/glossary.md#validator-node) 作为Libra生态系统的一部分，提供判定哪些交易可以被添加到Libra区块链上参与[共识](reference/glossary.md#consensus-protocol)。
验证器使用共识协议有容纳恶意节点的容错能力。在内部，[Validator nodes](reference/glossary.md#validator-node)需保持现有状态以执行交易及计算下一状态。[交易生命周期](life-of-a-transaction).一节将提供更多关于[Validator nodes](reference/glossary.md#validator-node)组件的信息。
 
测试网络是一组公开的Validator节点，可用于试用本系统。也可以试用Libra Core自行运行验证器节点。

### 开发者

Libra生态系统支持各种不同的开发者，不论是对Libra Core有所贡献的人员还是使用区块链构建应用程序的人员。开发者可能的工作包括但不限于： 
* 构建Libra客户端；
* 构建与Libra客户端交互的应用程序；
* 编写在Libra区块链上运行的智能合约；
* 对Libra区块链软件做出贡献。

该网站为开发人员设计

## 参考

* [Libra Protocol: Key Concepts](libra-protocol.md) &mdash; Introduces you to the fundamental concepts of the Libra protocol.
* [My First Transaction](my-first-transaction.md) &mdash; Guides you through executing your very first transaction on the Libra Blockchain using the Libra CLI client.
* [Getting Started With Move](move-overview.md) &mdash; Introduces you to a new blockchain programming language called Move.
* [Life of a Transaction](life-of-a-transaction.md) &mdash; Provides a look at what happens "under the hood" when a transaction is submitted and executed.
* [Libra Core Overview](libra-core-overview.md) &mdash; Provides the concept and implementation details of the Libra Core components through READMEs.
* [CLI Guide](reference/libra-cli.md) &mdash; Lists the commands of the Libra CLI client and their usage.
* [Libra Glossary](reference/glossary.md) &mdash; Provides a quick reference to Libra terminology.

翻译：Jadris Lau 校对：Zhe Wang