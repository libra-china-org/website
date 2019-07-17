---
id: libra-protocol
title: Libra协议： 核心概念
---


Libra区块链是一个基于Libra协议的加密认证的分布式数据库。本文将简略介绍Libra协议的核心概念。其详细说明请参阅[Libra技术白皮书](https://libra.org/en-us/whitepaper).

Libra区块链由分布式的[Validator节点网络](reference/glossary.md#validator-node)维护, 或简称为Validator。Validator集体遵循[共识协议](reference/glossary.md#consensus-protocol) 决定区块链中交易的进行次序。 

Libra测试网络是Libra区块链项目早期原型，即Libra Core的Demo 。 

## 交易和状态

Libra协议的两个核心基本概念为交易和状态在任一时间点，区块链都有一个所谓的状态。状态(或成为分布式账本状态)表示区块链上数据此时的快照。交易的执行会改变区块链的状态。 

![图 1.1 A 交易状态变更.](assets/illustrations/transactions.svg)
<center> <small class="figure">图 1.1 交易状态变更.</small> </center>

图 1.1 展示了执行交易时，Libra区块链的状态变化。例如，在状态 S~N-1~ 时，Alice 的余额为110 Libra币, Bob的余额为52 Libra币.交易发生后，区块链生成一个新的状态。在状态 S~N-1~ 的前提下，交易 T~N~ 发生，则状态由 S~N-1~ 变更为 S~N~ 。Alice的余额减少10Libra币，Bob的余额增加了10Libra币新的状态S~N~ 展示了状态更新后的账户余额情况。在图1.1中:

* **A** and **B** 分别代表Alice和Bob在区块链上的账户。
* **S~N-1~** 代表区块链中第(N-1)个状态。
* **T~N~** 代表区块链中执行的第N个交易。  
    * 从图中的例子可以看出，T~N~ 代表的交易是：从A账户中转10 Libra币到B账户中。
* **F**为一个确定性函数。在特定的初始状态执行特定的交易,F函数总会返回相同的最终状态。如果当前区块链状态为 S~N-1~, 执行交易 T~N~ ，则返回的新状态恒为 S~N~ 。
* **S~N~** 代表区块链的第N个状态。S~N~ 为将函数F应用于 S~N-1~ 和 T~N~ 的结果。

Libra协议使用 [Move 语言](move-overview.md) 来实现函数F的确定性执行。

### 交易

Libra区块链客户端通过提交交易请求来更新分布式账本状态。区块链上一个签名交易包括：

* **发送方地址** &mdash; 交易发起者的账户地址。
* **发送方公钥** &mdash; 用于签署交易的私钥所对应的公钥。
* **程序** &mdash; 程序包括以下内容：
    * 一个Move语言的字节码交易脚本； 
    * 可选的输入列表：在点对点交易中，输入包括接受者信息及金额；
    * 可选的Move字节码模块部署列表； 
* **Gas价格** (以microlibra/gas 为单位&mdash;执行交易时，发送方愿意为一单位[gas](reference/glossary.md#gas) 所支付的价格。Gas是用来支付在区块链上计算和存储费用。每一Gas单位是对计算量的抽象度量；
* **Gas上限** &mdash; 交易允许消耗的Gas最大值；
* **序号** &mdash; 无符号整型，必须和发送者账户中的序列号相等；
* **有效期** &mdash; 交易的有效截止时间；
* **签名** &mdash; 发送者的数字签名。

交易脚本是任意包含对交易逻辑编码的程序，能够与Libra区块链中发布的数字资产进行交互。 

### 分布式账本状态

分布式账本状态，又称为Libra区块链全局状态，是区块链上所有账户状态的集合。想要执行交易，每个Validator必须获得区块链上分布式数据库的最新全局状态。更多见[版本化数据库](#versioned-database).

## 版本化数据库

Libra区块链上的所有数据都存储在一个单一版本化的分布式数据库上。版本号为无符号的64位整数，与系统内已经执行的交易数量相对应。

版本化数据库允许Validator：

* 在最新的全局状态下进行交易；
* 响应客户端发送的对当前或历史全局状态的请求。

## 账户

Libra账户包括Move模块和Move资源。由[账户地址](reference/glossary.md#account-address)标识。这也意味着每个账户的状态都包含代码和数据两方面。 

* **[Move模块](move-overview.md#move-modules-allow-composable-smart-contracts)** 包含代码(类型和过程声明),但不包含数据。模块中的子程序对区块链全局状态的更新规则进行编码。
* **[Move资源](move-overview.md#move-has-first-class-resources)** 包含数据不包含代码。每个资源的类型都应在区块链的分布式数据库中已发布模块里声明过。

账户可以包含任意数量的Move资源和Move模块。

#### 账户地址

Libra账户地址为一个256位的值。用户可以使用电子签名来声明地址。对于一个账户，其地址由用户的公钥通过密码学Hash（或托管的客户端）生成，用户必须通过相应的私钥签名才能从此账户发起交易。

Libra对用户的账户地址的数量不做限制。但申请新账户地址时，必须通过另一Libra币充足的账户支付申请费用。

## 证明

Libra区块链上的所有数据都存储在一个单一版本化的分布式数据库上，存储被用来对交易区块和交易结果的持续确认。区块链是一个不断增长的[Merkle交易树](reference/glossary.md#merkle-trees). 每次区块链上有新的交易执行，交易树都会增加一片“叶子”。

* 证明是验证Libra区块链中数据真实性的一种方式； 
* 区块链上存储的每个操作都可以进行加密验证，结果性证明也可以证实没有数据缺损。例如，如果客户端发送了对最新的 _n_ 笔交易的查询请求，证明可以验证查询响应中没有遗漏任何一笔交易记录。

在区块链中，客户端不需要信任接受数据的实体。客户端可以查询账户余额，以及特定交易的交易状态等。与其他Merkle树类似，分布式账本记录可以对特定的交易提供 $O(\log n)$ 时间复杂度证明， _n_ 为处理的交易总量。

## Validator 节点 (validator)

Libra区块链的客户端创建交易并提交到Validator节点。Validator节点（和其他Validator节点共同）运行共识协议，执行交易，并将交易和执行结果存储在区块链中。Validator节点判定哪些交易可以被添加到区块链上，以及以何种次序添加。
![图 1.1 validator逻辑组件](assets/illustrations/validator.svg)
<small class="figure">图 1.2 Validator逻辑组件</small>

 Validator节点 包括以下逻辑组件：

**准入控制 (AC)**

* 准入控制是Validator节点的唯一外部接口。客户端向Validator节点发送的任何请求都将先转入AC； 
* 准入控制通过对请求进行初始检查，来保护Validator节点的其他部件免受损坏或大量输入的影响。

**内存池**

* 内存池是一个缓存区，用于保存“等待”执行的交易。 
* 当一个内存池中添加了新交易时，这个内存池会和系统中其他Validator节点的内存池共享此交易。 

**共识**

* 共识组件负责判断交易区块的顺序，并与区块链中其他的Validator节点在[共识协议](reference/glossary.md#consensus) 下共同决定执行结果，

**执行**

* 执行组件利用虚拟机（VM）交易。
* 执行组件的作用是协调一个区块中的交易的执行，并维护一个可用于协商投票的瞬时状态;
* 执行组件维护内存中执行结果，直至共识组件允许其被提交到分布式数据库。

**虚拟机 (VM)**

* 准入控制和内存池借助虚拟机组件对交易进行校验。
* 虚拟机用于运行交易中所包括的程序，并确定结果。

**存储**

存储被用来持久化保存已确认的交易区块和交易结果；

更多Validator组件与其他组件的交互信息，请参考[交易生命周期](life-of-a-transaction.md).

## 参考：

* [欢迎页](welcome-to-libra.md).
* [My First Transaction](my-first-transaction.md) &mdash; Guides you through executing your very first transaction on the Libra Blockchain using the Libra CLI client.
* [Getting Started with Move](move-overview.md) &mdash; Introduces you to a new blockchain programming language called Move.
* [Life of a Transaction](life-of-a-transaction.md) &mdash; Provides a look at what happens “under the hood” when a transaction is submitted and executed.
* [Libra Core Overview](libra-core-overview.md) &mdash; Provides the concept and implementation details of the Libra Core components through READMEs.
* [CLI Guide](reference/libra-cli.md) &mdash; Lists the commands (and their usage) of the Libra CLI client.
* [Libra Glossary](reference/glossary.md) &mdash; Provides a quick reference to Libra terminology.

翻译：Jadris Lau 校对：Zhe Wang