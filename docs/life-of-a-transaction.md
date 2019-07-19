---
id: life-of-a-transaction
title: 交易生命周期
---

为了更加深入的理解Libra的交易生命周期，我们将跟随一个交易的全过程，从其被提交到Libra validator始，直至其被添加到区块链上止。我们将“放大”来看每个validator逻辑组件及与其他组件之间的交互。

## 客户端提交交易

Libra客户端构造 **原始交易** (此处称为T~5~raw)，从Alice的账户中转移10Libra币到Bob的账户中。原始交易应包含以下字段：每个字段都通过超链接关联到词汇定义表。

* Alice的[账户地址](reference/glossary.md#account-address).
* 一个表明Alice方将执行的操作的程序，包括:
    * 一个Move [点对点字节码交易脚本](reference/glossary.md#transaction-script).
    * 脚本输入参数列表 (在这个例子中为Bob的账户地址和付款金额等).
* [Gas价格](reference/glossary.md#gas-price) (以microlibra/gas为单位) &mdash; Alice愿意为执行本次交易所需的每单位Gas支付的价格。Gas用于支付区块链上的计算和存储费用。每一Gas单位是对计算量的抽象度量
* Alice愿意为为此次交易支付的[Gas上限](reference/glossary.md#maximum-gas-amount) 
* 此次交易的[有效期](reference/glossary.md#expiration-time) 
* [序号](reference/glossary.md#sequence-number) &mdash; 5
    * 序号为5的交易，只能在包含5个交易的账户中发起。

客户端使用Alice的私钥对交易T~5~raw **签名** 。签名后的交易T~5~ 包括下列内容：

* 原始交易
* Alice的公钥
* Alice的数字签名

### 交易前的假设

为了描述交易T~5~的生命周期, 我们有如下假设:

* Alice和Bob在Libra区块链上都拥有[账户](reference/glossary.md#accounts)。
* Alice账户中有110Libra币。
* Alice账户中当前的[序列号](reference/glossary.md#sequence-number) 是5 (表示Alice的账户已经发送了5次交易).
* 网络中共有100个validator&mdash;从 V~1~ 到 V~100~ 。
* 客户端将交易T~5~ 提交给validator V~1~
* **Validator V~1~ 即为本轮共识的倡议者或领导者**

## 交易的生命周期

在本节中我们将讨论交易T~5~的生命周期, 从其被提交到Libra validator始，直至其被添加到区块链上止。

下图展示了validator 节点相关组件之间的交互链的相关节点。在熟悉交易生命周期的所有步骤后，你可以进一步了解各步骤中相应组件的交互信息。

<blockquote class="block_note">

**注意:**本图标中的所有箭头都起始于发起交互或操作的组件，终止于执行操作的组件。此处箭头 **不表示** 数据的读取，写入或返回。
</blockquote>

![Figure 1.1 交易生命周期](assets/illustrations/validator-sequence.svg)
<small class="figure">Figure 1.1 交易生命周期</small>

### 接收交易

**1** &mdash; 客户端将交易T~5~ 提交给validator V~1~ ，V~1~ 的准入控制组件（AC）接受该交易。(客户端 → AC [AC.1](#client-ac-ac1))

**2** &mdash; AC 利用虚拟机组件 (VM) 执行查验，如签名验证，检查Alice账户余额是否充足，检查交易V~1~ 是否 被重复广播等(AC → VM [AC.2](#ac-vm-ac2), [VM.1](#ac-vm-vm1))

**3** &mdash; 当交易T~5~ 通过查验后， AC 将交易T~5~ 发送到to V~1~’的内存池(AC → 内存池 [AC.3](#ac-mempool-ac3), [MP.1](#ac-mempool-mp1))

### 发布交易到其他Validators

**4** &mdash; 内存池将交易T~5~ 保存在内存缓冲区中。内存池内可能已经包含了从Alice账户地址发送的多个交易。

**5** &mdash; 利用内存池共享协议, V~1~ 将发送其内存池中所有交易(包括交易T~5~) 给其他validator(V~2~ to V~100~) 并将从其他validator接受到的交易存储在自身的内存池中(Mempool →其他 Validators [MP.2](#mempool-other-validators-mp2))

### 交易区块发起

**6** &mdash; 作为本轮共识发起者V~1~ 将从其内存池中提取一个区块，并通过共识组件复制分发给其他的validator。(共识组件 → 内存池[MP.3](#consensus-mempool-mp3), [CO.1](#consensus-mempool-co1))

**7** &mdash; V~1~ 的共识组件负责协调所有validator发送的区块中交易的顺序。(共识 → 其他Validators [CO.2](#consensus-other-validators-co2)).关于此处提出的共识协议LibraBFT,请参阅技术性文献[Libra区块链中的状态机复制](state-machine-replication-paper.md) 

###区块的执行和共识的达成

**8** &mdash; 作为共识达成过程的一个步骤，交易区块(包括交易 T~5~) 被提交给执行组件。(共识 → 执行[CO.3](#consensus-execution-consensus-other-validators-co3), [EX.1](#consensus-execution-ex1))

**9** &mdash; 执行组件通过虚拟机(VM)管理交易的执行。应注意，此处的执行是在达成共识前的推测性执行。(执行→ VM [EX.2](#execution-vm-ex2), [VM.3](#execution-vm-vm3))

**10** &mdash; 在上述执行完成后，执行组件将交易区块将(包含T~5~)附加到 (分布式账本的历史)[Merkle累加器](#merkle-accumulators) 上，形成内存/临时版本Merkle累加器。执行（发起和推测）交易的结果会返回到共识组件。(共识 → 执行[CO.3](#consensus-execution-consensus-other-validators-co3), [EX.1](#consensus-execution-ex1)).从共识指向执行的箭头表示该执行请求由共识组件发起。(为了减少歧义，本文中将不使用箭头表示数据流).

**11** &mdash; （共识发起者）V~1~ 试图与其他共识参与者，即其他validator就该区块的执行结果达成共识。(共识→ 其他 Validators [CO.3](#consensus-execution-consensus-other-validators-co3))

###区块的提交

**12** &mdash; 当对区块的执行结果达成共识并签名的validator数量上具有绝对优势时，V~1~'的执行组件将从缓存中读取区块的推测性执行结果，并将区块中的所有交易提交到永久存储中。(共识 → 执行[CO.4](#consensus-execution-co4), [EX.3](#consensus-execution-ex3)), (Execution → Storage [EX.4](#execution-storage-ex4), [ST.3](#execution-storage-st3))

**13** &mdash; Alice账户现在的余额为100Libra币，序列号为6.如T~5~ 被Bob重复广播，该交易将被拒绝，因为Alice账户的序列号（6）大于被重复广播的交易的序号（5）

## Validator 组件交互

在[上一部分](#lifecycle-of-the-transaction)中, 我们举例说明了一个典型的交易生命周期，从其被提交到Libra validator始，直至其被添加到区块链上止。现在我们将更加深入的探究validator处理交易并响应请求时，组件间的交互。对下述人员，这些信息可能十分值得参考：

* 想要全面了解底层系统是如何运作的；
* 有兴趣为Libra Core软件做出贡献的

为叙述方便，我们假设客户端提交一个交易T~N~ 给一个validatorV~X~.对每个validator组件，我们将在其相对应子章节中具体描述组件之间的交互。此处应注意，用以描述组件间交互的子章节顺序并未严格按照其执行顺序。组件间的大部分交互都与交易的执行有关，少数与客户端发出的（查询区块链上的已知信息）请求有关。

 我们先来看validator 节点的核心逻辑组件：

* [准入控制](#admission-control-ac)
* [内存池](#mempool)
* [共识](#consensus)
* [执行](#execution)
* [虚拟机](#virtual-machine-vm)
* [存储](#storage)

每一小节结尾都附有[Libra Core](libra-core-overview.md)中的相应文档。

## 准入控制(AC)

![Figure 1.2 准入控制(assets/illustrations/admission-control.svg)
<small class="figure">Figure 1.2 准入控制</small>

准入控制是validator 的_唯一外部接口_。客户端向Validator节点发送的任何请求都将先转入AC。

### 客户端→ AC (AC.1)

一个客户端将交易提交给V~X~的准入控制组件。此步骤通过:
`AC::SubmitTransaction()`完成。

### AC → VM (AC.2)

准入控制组件访问validator的虚拟机并对交易进行预查验，以便尽早拒绝格式错误的交易。此步骤
 [`VM::ValidateTransaction()`](#virtual-machine-b)完成。

### AC → 内存池(AC.3)

一旦`VM::ValidateTransaction()` 没有返回错误, AC 就通过`Mempool::AddTransactionWithValidation().`将交易转发到V~X~的内存池。当且仅当T~N的序号大于或等于发送者账户的序号时，V~X~的内存池才会接受来自AC的交易T~N（注意：在交易序号和账户的下一个序号相等前，交易无法达成共识）

### AC → 存储(AC.4)

当客户端对Libra区块链执行查询时（例如获取Alice账户余额）。AC直接与存储组件交互，以获取所请求的信息。

### 准入控制文档

更多实现细节，请参阅[准入控制](crates/admission-control.md)文档。

##虚拟机(VM)

![Figure 1.3 Virtual Machine](assets/illustrations/virtual-machine.svg)
<small class="figure">Figure 1.3 虚拟机</small>

[Move虚拟机](move-overview.md) (VM) 验证并执行Move字节码编写的脚本。

### AC → VM (VM.1)

V~X~的准入控制组件接收到交易请求时， 将调用虚拟机上的`VM::ValidateTransaction()` 程序验证交易。

### VM → 存储 (VM.2)

当准入控制组件或内存池通过`VM::ValidateTransaction()`, 向虚拟机提交验证交易请求时虚拟机加载存储中保存的交易发起者账户，并执行以下验证：

* 检查交易中的签名是否正确（并拒绝签名错误的交易）；
* 验证发起者的账户身份验证密钥是否与其（签署交易私钥对应的）公钥的哈希值一致；
* 核查交易的序号是否不小于发起者账户当前的序列号。  此检查可防止发起者账户发出的交易被重复执行；
* 验证签名交易的程序代码是否有格式错误，虚拟机无法执行格式错误的程序代码；
* 验证发起者账户是否有充足的余额支付专用于此交易的Gas上限，以确保交易有支付资源使用费的能力。

### 执行 → 虚拟机 (VM.3)

执行组件通过调用虚拟机的`VM::ExecuteTransaction()`函数执行交易。

此处应重点注意，执行交易不同于更新分布式账本状态或存储执行结果。首先，在共识期间，执行交易T~N~ 以获得区块的共识。当与其他validator就交易顺序和执行结果达成共识后，执行结果才被永久存储，分布式账本状态也随之更新。

### 内存池 → 虚拟机(VM.4)

当内存池通过其他validator共享内存池接收到交易请求时，内存池随即调用虚拟机上的[`VM::ValidateTransaction()`](#action-b-1)来验证交易

### 虚拟机README

更多实现细节，请参阅 [虚拟机文档](crates/vm.md).

## 内存池

![Figure 1.4 内存池](assets/illustrations/mempool.svg)
<small class="figure">Figure 1.4 内存池</small>

内存池是一个共享缓冲区，用于保存等待执行的交易。当一笔新交易添加到内存池时，内存池与系统的其他validator共享此交易。为减少网络消耗，在共享的内存池系统中，每个validator仅负责传递自己的交易，当validator从其他validator内存池发送的交易时，该交易将会被添加到接受者的内存池中。

### AC → 内存池(MP.1)

* 执行初始查验后，validator的准入控制组件将交易发送到其内存池；
* 当且晋档收件人账户发来的交易T~N~的序号大于或等于发件人账户的序列号时，V~X~的内存池才接收此交易。

### 内存池→其他Validators (MP.2)

* V~X~ 的内存池将交易T~N~ 共享给同一网络中的其他validator；
*其他validators将其内存池中的交易共享给 V~X~的内存池；（即相互共享内存池中的交易）

###共识→内存池(MP.3)

* 当 V~X~ 为共识发起者时，其共识组件将从其内存池中提取一个交易区块并复制转发给其他validator，以就交易顺序和交易结果达成共识。
* 此处注意，尽管T~N~ 此时在交易区块中，但 T~N~ 并不一定最终能被永久存储在区块链的分布式数据库中。

### 内存池 → 虚拟机(MP.4)

当内存池接收到其他validator发送的交易时，将在虚拟机上调用[`VM::ValidateTransaction()`](#action-b-1)来验证交易。

### 内存池README

更多实现细节，请参阅[内存池文档](crates/mempool).

## 共识

![Figure 1.5 共识](assets/illustrations/consensus.svg)
<small class="figure">Figure 1.5 共识</small>

共识组件用于与网络中参与[共识协议](#consensus-protocol)其他validator就交易区块结果和顺序达成共识 

### 共识 → Mempool (CO.1)

当 V~X~ 为交易发起者时 V~X~ 调用`Mempool::GetBlock()`, 从其内存池中提取交易区块并发起提议。

### 共识→ 其他Validators (CO.2)

若V~X~ 为共识发起者，其共识组件将所提取的交易区块复制转发给其他validator

### 共识 → 执行, 共识 → 其他Validators (CO.3)

* 交易区块内的执行要求共识和其他组件进行交互。共识通过调用`Execution:ExecuteBlock()` (参考 [共识 → 执行](#consensus-execution-ex1))执行交易；
* 区块内的交易执行后，执行组件返回交易的执行结果；
*共识签署执行结果，并试图与参与协商的其他validator达成共识。

### 共识 → 执行(CO.4)

当对区块的执行结果达成共识的validator数量上具有绝对优势时，V~X~ 的共识组件通过`Execution::CommitBlock()` 通知执行组件该交易块已准备好被提交。

### 共识README

更多实现细节，请参阅[共识README](crates/consensus.md).

## 执行

![Figure 1.6 执行](assets/illustrations/execution.svg)
<small class="figure">Figure 1.6 执行</small>

执行组件用于协调交易区块的执行以及维持可供共识协商的瞬时状态。

### 共识 → 执行(EX.1)

*  共识通过`Execution::ExecuteBlock()`请求执行组件执行交易区块。
* 执行组件维护一个“暂存器”，该暂存器用以保存内存中[Merkle 累加器](#merkle-accumulators)相关部分的副本；此信息用以计算区块链当前状态的根哈希。
* 根哈希和区块中交易信息共同确定累加器的新根哈希。此步骤在永久保存新数据前执行，用以确保在获得共识前，所有交易或状态都不被存储。
* 执行组件计算推测性执行的根哈希后， V~X~ 的共识组件签署并试图就此根哈希与其他validator达成共识。

### 执行 → 虚拟机(EX.2)

当共识组件调用`Execution::ExecuteBlock()`请求执行组件执行一个交易区块时， 执行组件调用虚拟机确定交易区块的执行结果。

### 共识→  (EX.3)执行

当对区块的执行结果达成共识的validator数量上具有绝对优势时，V~X~ 的共识组件通过`Execution::CommitBlock()` 通知执行组件该交易块已准备好被提交。  此请求包含签署共识的validator的签名作为其同意共识的证明。

### 执行 → 存储(EX.4)

执行组件从其暂存器提取值，并通过`Storage::SaveTransactions()`将值发送到存储以进行永久保存。之后，执行组件会删除无用值(如因重复无法提交的区块)。

### 执行README

更多实现细节，请参阅 [执行README](crates/execution).

## 存储

![Figure 1.7 存储](assets/illustrations/storage.svg)
<small class="figure">Figure 1.7 存储</small>

存储组件用以永久性保存交易区块及其执行结果。在下列情况下，一个交易区块/交易组(包括 T~N~)将被永久保存：

* 网络中超过2f+1的validators 就以下几点达成共识：
    * 区块中包含的交易；
    * 区块中交易的顺序；
    * 区块中交易的执行结果。

有关如何将交易附加到区块链上的数据结构信息，请参阅[Merkle 累加器](reference/glossary.md#merkle-accumulators) 

### 虚拟机 →存储(ST.1)

当准入控制或内存池调用`VM::ValidateTransaction()` 来验证交易时，`VM::ValidateTransaction()` 加载存储中的发送人账户，并通过只读模式检查交易的有效性。

### 执行 → 存储(ST.2)

当共识组件调用`Execution::ExecuteBlock()`, 执行组件读取当前状态和内存中的暂存器数据来确定执行结果。

### 执行 → 存储(ST.3)

* 一旦交易共识达成，执行组件调用`Storage::SaveTransactions()` 来保存交易区块并永久保存相关记录。同时，网络中签署共识的validator的签名也将被保存；
* 暂存器的内存数据将更新到存储中并永久保存交易。
* 存储更新后，所有参与交易的资源的序列号都会相应更新；
* 注意: Libra区块链账户的序列号随着交易数量等量增加。

### AC → 存储(ST.4)

当客户方发送对区块链中信息的查询请求时，准入控制组件和存储直接交互以读取所相应信息。

### 存储README

更多实现细节，请参阅 [存储README](crates/storage.md).

## 参考：

* [Welcome page](welcome-to-libra.md).
* [Libra Protocol: Key Concepts](libra-protocol.md) &mdash; Introduces you to the fundamental concepts of the Libra protocol.
* [My First Transaction](my-first-transaction.md) &mdash; Guides you through executing your very first transaction on the Libra Blockchain using the Libra CLI client.
* [Getting Started With Move](move-overview.md) &mdash; Introduces you to a new blockchain programming language called Move.
* [Libra Core Overview](libra-core-overview.md) &mdash; Provides the concept and implementation details of the Libra Core components through READMEs.
* [CLI Guide](reference/libra-cli.md) &mdash; Lists the commands (and their usage) of the Libra CLI client.
* [Libra Glossary](reference/glossary.md) &mdash; Provides a quick reference to Libra terminology.
* [State Machine Replication in the Libra Blockchain](state-machine-replication-paper.md) &mdash; Provides a detailed look into our consensus protocol **LibraBFT**.

翻译：Jadris Lau 校对：Zhe Wang