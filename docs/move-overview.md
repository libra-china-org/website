---
id: move-overview
title: Move语言初体验
---

Move是一种全新的编程语言，专门为Libra区块链设计，以提供安全且程式化的基础。Libra区块链上的账户，包含任意数量的Move资源和Move模块。提交给Libra区块链的每一笔交易请求，都应使用Move语言编写的交易脚本。交易脚本可以调用模块所声明的过程，以更新区块链的全局状态。

本指南第一部分将从总体的层面面介绍Move语言的

1.[程式化交易利用Move语言编写的交易脚本实现](#move-transaction-scripts-enable-programmable-transactions)

2.[Move Modules Allow Composable Smart Contracts](#move-modules-allow-composable-smart-contracts)

3.[Move](#move-has-first-class-resources)

若您对此兴趣浓厚，可进一步参看[Move技术手册](move-paper.md)以了解更加详细的信息。

本指南的第二部分则会深入展示如何在[Move intermediate representation](#move-intermediate-representation)中编写Move程序。初始版本的测试网络中不支持自定义Move程序，但此功能可在本地试用。

## Move语言特性

### Move语言可实现可编程性高的交易脚本

* 每一个Libra上的交易都含有**Move 交易脚本** ，对validator代表用户端的行为逻辑进行编码。（如从Alice账户中转Libra币到Bob账户中） 
* 交易脚本通过调用一个或多个[Move模块](#move-modules-allow-composable-smart-contracts)，与Libra区块链上公开在全局存贮的[Move资源](#move-has-first-class-resources)进行交互。 
* 交易脚本不存储在区块链全局状态中，也无法被其他交易脚本调用，只能使用一次。
* 在[编写交易脚本](#writing-transaction-scripts)中，提供了几个交易脚本示例。

### Move Modules Allow Composable Smart Contracts

Move模块定义区块链全局状态的更新规则。Move模块和其他区块链的智能合约具有相同优势。模块声明[resource](#move-has-first-class-resources)用户账户可发布的资源类型。Libra区块链上的每一个账户都包含任意数量的模块和资源。

* 模块既可声明结构类型（资源为一种特殊的结构类型），也可声明过程。
* Move模块声明的过程，定义了创建，访问和销毁其所声明类型的规则。
* 模块可重复调用一个模块中声明的结构类型可以被另一个模块使用，一个模块中声明的过程也能引发另一个模块声明过程。模块也会调用其他模块中声明的过程。交易脚本可调用已发布模块的任意公开过程。
* Libra用户可以用自己的账户发布模块。

### Move具有一流的资源

* Move的特征之一就是它能自定义资源类型资源类型用于编码高可编程性的安全数字资产。
*资源为Move语言中的常规值，可被存储为数据结构，可被作为参数传递，也可被过程返回，等等。 
* Move编码系统为资源提供了针对性的安全保护。Move资源不可复制，重复使用或销毁。一个资源类型仅能被定义其类型的模块创建或销毁。[Move虚拟机](reference/glossary.md#move-virtual-machine-mvm) 将通过静态字节码验证，并拒绝未通过字节码验证的的程序运行，以此确保其安全性。
* Libra货币 作为资源类型`LibraCoin.T`被使用。`LibraCoin.T` 不享受特殊地位，所有Move资源享有相同的保护机制。

## Move: 深入了解

### Move的中间表示（IR）

此部分介绍如何利用IR表示编写[交易脚本](#writing-transaction-scripts)和[modules](#writing-modules)。请特别注意，Move IR是将要推出的Move语言的先行版本，此版本仍不稳定([开发展望](#future-developer-experience) 中提供了更多相关细节)。Move IR 仅是Move字节码的句法层面展示，用以测试字节码验证器和虚拟机，开发者友好度并不高。它能够完成可读代码的编译，也可直接编译底层Move字节码。尽管Move IR仍存在诸多不足，但我们希望各位开发人员能试试看。 

以下内容为Move IR的一些程序示例。我们鼓励大家跟随下列示例，在本地编译，运行，调试自定义程序。`libra/language/README.md`和`libra/language/compiler/README.md`目录下的Read Me 是对如何执行操作的说明

### 编写交易脚本

As we explained in [Move Transaction Scripts Enable Programmable Transactions](#move-transaction-scripts-enable-programmable-transactions)用户可编写交易脚本以发送对Libra区块链全局状态的更新请求。 几乎所有交易脚本都会包括两个重要模块： `LibraAccount.T` 资源型和`LibraCoin.T` 资源型。`LibraAccount` 为模块名`T` 为模块声明的资源类型。这是Move语言的常用命名规则， 模块声明的“最主要”类型一般命名为`T`。 

当我们说某个用户有`0xff` 为Libra区块链账户时，实际意味着 地址`0xff` 存储着`LibraAccount.T`资源实体。每个不为空的地址都有相应的 `LibraAccount.T` 资源。此资源保存了账户信息，如序列号，身份验证密钥及余额等。Libra区块链上的人任意部分，若想和一个账户交互，必须先读取`LibraAccount.T` 或调用`LibraAccount` 模块。

账户余额是`LibraCoin.T`类型的资源。如同[Move Has First Class Resources](#move-has-first-class-resources)所提及，这是Libra币的资源类型。此类型的资源享受其他Move资源的相同的地位。`LibraCoin.T`类型的资源可被作为程序变量存储，可被在过程间传递，等等。

我们鼓励感兴趣的读者在在`libra/language/stdlib/modules/` 目录下检索 `LibraAccount` 和`LibraCoin` 模块的资源类型。

现在让我们如何编译交易脚本来实现这些模块和资源的交互。

```move
// Simple peer-peer payment example.

// Use LibraAccount module published on the blockchain at account address
// 0x0...0 (with 64 zeroes).0x0 is shorthand that the IR pads out to
// 256 bits (64 digits) by adding leading zeroes.
import 0x0.LibraAccount;
import 0x0.LibraCoin;
main(payee: address, amount: u64) {
  // The bytecode (and consequently, the IR) has typed locals.  The scope of
  // each local is the entire procedure.All local variable declarations must
  // be at the beginning of the procedure.Declaration and initialization of
  // variables are separate operations, but the bytecode verifier will prevent
  // any attempt to use an uninitialized variable.
  let coin: R#LibraCoin.T;
  // The R# part of the type above is one of two *kind annotation* R# and V#
  // (shorthand for "Resource" and "unrestricted Value").These annotations
  // must match the kind of the type declaration (e.g., does the LibraCoin
  // module declare `resource T` or `struct T`?).

  // Acquire a LibraCoin.T resource with value `amount` from the sender's
  // account.  This will fail if the sender's balance is less than `amount`.
  coin = LibraAccount.withdraw_from_sender(move(amount));
  // Move the LibraCoin.T resource into the account of `payee`.If there is no
  // account at the address `payee`, this step will fail
  LibraAccount.deposit(move(payee), move(coin));

  // Every procedure must end in a `return`.The IR compiler is very literal:
  // it directly translates the source it is given.It will not do fancy
  // things like inserting missing `return`s.
  return;
}
```

此脚本有一个缺陷 &mdash; 若 `payee`地址下没有账户，此程序将不能运行。我们可以将脚本修改为若 `payee` 地址下没有账户，则为其创建账户，来解决这个问题。

```move
// A small variant of the peer-peer payment example that creates a fresh
// account if one does not already exist.

import 0x0.LibraAccount;
import 0x0.LibraCoin;
main(payee: address, amount: u64) {
  let coin: R#LibraCoin.T;
  let account_exists: bool;

  // Acquire a LibraCoin.T resource with value `amount` from the sender's
  // account.  This will fail if the sender's balance is less than `amount`.
  coin = LibraAccount.withdraw_from_sender(move(amount));

  account_exists = LibraAccount.exists(copy(payee));

  if (!move(account_exists)) {
    // Creates a fresh account at the address `payee` by publishing a
    // LibraAccount.T resource under this address.If theres is already a
    // LibraAccount.T resource under the address, this will fail.
    create_account(copy(payee));
  }

  LibraAccount.deposit(move(payee), move(coin));
  return;
}
```

现在来看一个更为复杂的例子。我们将编译一个交易脚本，实现对多个接受者的支付。

```move
// Multiple payee example.This is written in a slightly verbose way to
// emphasize the ability to split a `LibraCoin.T` resource.The more concise
// way would be to use multiple calls to `LibraAccount.withdraw_from_sender`.

import 0x0.LibraAccount;
import 0x0.LibraCoin;
main(payee1: address, amount1: u64, payee2: address, amount2: u64) {
  let coin1: R#LibraCoin.T;
  let coin2: R#LibraCoin.T;
  let total: u64;

  total = move(amount1) + copy(amount2);
  coin1 = LibraAccount.withdraw_from_sender(move(total));
  // This mutates `coin1`, which now has value `amount1`.
  // `coin2` has value `amount2`.
  coin2 = LibraCoin.withdraw(&mut coin1, move(amount2));

  // Perform the payments
  LibraAccount.deposit(move(payee1), move(coin1));
  LibraAccount.deposit(move(payee2), move(coin2));
  return;
}
```

对交易脚本编写的介绍到此结束。更多相关示例，包括初始版本测试网络支持的交易脚本，请参阅`libra/language/stdlib/transaction_scripts`。

### 编写模块

下面将介绍如何编写自己的Move模块，而不是仅仅重复使用现有的 `LibraAccount` 和`LibraCoin` 模块。想象下这种情况：
未来某天，Bob可能会在地址 *a* 上创建一个账户。Alice现在想指定一部分资金，当Bob账户创建后转账给Bob。但她也希望，如果Bob不创建账户，她能拿回自己的资金。

为了帮助Alice解决这个问题，我们编写`EarmarkedLibraCoin` 模块，此模块
* 声明新资源类型`EarmarkedLibraCoin.T` ，包括Libra币和接受者账户地址。
* 允许Alice创建及使用账户发布某种类型的资源( `create` 过程)。
* 允许Bob要求资源(`claim_for_recipient` 过程)。
*允许任何拥有`EarmarkedLibraCoin.T` 资源类型的人销毁并取回冻结的资金( `unwrap` 过程)。

```move
// A module for earmarking a coin for a specific recipient
module EarmarkedLibraCoin {
  import 0x0.LibraCoin;

  // A wrapper containing a Libra coin and the address of the recipient the
  // coin is earmarked for.
  resource T {
    coin: R#LibraCoin.T,
    recipient: address
  }

  // Create a new earmarked coin with the given `recipient`.
  // Publish the coin under the transaction sender's account address.
  public create(coin: R#LibraCoin.T, recipient: address) {
    let t: R#Self.T;

    // Construct or "pack" a new resource of type T. Only procedures of the
    // `EarmarkedLibraCoin` module can create an `EarmarkedLibraCoin.T`.
    t = T {
      coin: move(coin),
      recipient: move(recipient),
    };

    // Publish the earmarked coin under the transaction sender's account
    // address.Each account can contain at most one resource of a given type; 
    // this call will fail if the sender already has a resource of this type.
    move_to_sender<T>(move(t));
    return;
  }

  // Allow the transaction sender to claim a coin that was earmarked for her.
  public claim_for_recipient(earmarked_coin_address: address): R#Self.T {
    let t: R#Self.T;
    let t_ref: &R#Self.T;
    let sender: address;

    // Remove the earmarked coin resource published under `earmarked_coin_address`.
    // If there is no resource of type T published under the address, this will fail.
    t = move_from<T>(move(earmarked_coin_address));

    t_ref = &t;
    // This is a builtin that returns the address of the transaction sender.
    sender = get_txn_sender();
    // Ensure that the transaction sender is the recipient.If this assertion
    // fails, the transaction will fail and none of its effects (e.g.,
    // removing the earmarked coin) will be committed.  99 is an error code
    // that will be emitted in the transaction output if the assertion fails.
    assert(*(&move(t_ref).recipient) == move(sender), 99);

    return move(t);
  }

  // Allow the creator of the earmarked coin to reclaim it.
  public claim_for_creator(): R#Self.T {
    let t: R#Self.T;
    let sender: address;

    sender = get_txn_sender();
    // This will fail if no resource of type T under the sender's address.
    t = move_from<T>(move(sender));
    return move(t);
  }

  // Extract the Libra coin from its wrapper and return it to the caller.
  public unwrap(t: R#Self.T): R#LibraCoin.T {
    let coin: R#LibraCoin.T;
    let recipient: address;

    // This "unpacks" a resource type by destroying the outer resource, but
    // returning its contents.Only the module that declares a resource type
    // can unpack it.
    T { coin, recipient } = move(t);
    return move(coin);
  }

}
```

Alice可以通过创建交易脚本在自己的账户余额中为Bob留出指定资金，并调用`create`过程，应用于Bob地址*a*和`LibraCoin.T`资源类型。一旦地址 *a* 创建成功，Bob就可以从地址*a*发送交易请求以获得Libra币。这将调用`claim_for_recipient`来将结果传递给`unwrap`，并将返回的`LibraCoin`存储。如果Bob很久都没创建地址为*a*的账户，Alice想收回她的自己，她可以依次调用`claim_for_creator` `unwrap`来实现。

细心地读者可能已经注意到了，这个模块中的代码和`LibraCoin.T`的内部结构无关。可以轻松使用泛型编程实现(如 `resource T<AnyResource: R> { coin: AnyResource, ...}`).我们也正致力于为Move添加此类型的参数多态支持。

### 未来开发展望

不久之后，IR将达到更为的稳定状态，编译器和验证程序的用户友好性也会有所增加。另外，IR源的本地信息也将被追踪并发送至验证器，以便更好的Debug。但IR仍将作为测试Move字节码的工具。它应该是底层字节码的直观展示。为了测试的有效进行，IR编译器会允许一些编码错误，比如被字节码检验器拒绝或在编译器中运行失败。而用户友好型的源语言则会拒绝编译将在后续步骤中返回错误的代码。

未来我们将推出更高级的，旨在安全简洁的表达惯用语法和编程模式的Move源语言。  由于Move字节码是新语言，Libra区块链是新编译环境。我们对于其支持的惯用语法和编程模式仍在探索阶段。Move源语言仍处于早期开发阶段，具体发布时间仍无法确定。

翻译：Jadris Lau 校对：Zhe Wang