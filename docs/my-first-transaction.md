---
id: my-first-transaction
title: 第一笔交易！
---

本文档将指导您在Libra区块链上执行第一笔交易。在交易前，建议您先阅读下列文档以熟悉Libra生态系统和Libra协议的核心概念：

* [欢迎](welcome-to-libra.md)
* [Libra协议： 核心概念](libra-protocol.md)

我们提供了一个命令行界面(CLI) 客户端实现与区块链的交互。

## 前提假设：

本文档中的所有命令行都以如下几点为前提：

* 您使用的是Linux系统 (基于Red Hat或 Debian-based)或macOS 系统；
* 您可以稳定连接到互联网；
* 系统中已经安装`git` 
* macOS系统中已经安装Homebrew；
* Linux系统中已经安装`yum`o或 `apt-get`； 

## 提交交易的步骤

下述例子中，我们将下载Libra的必要组件并在两个用户（Alice和Bob）间执行一个交易。

执行下列步骤将交易提交到Libra测试网络上的validator：

1.[克隆并构建Libra Core](#clone-and-build-libra-core).
2.[构建Libra CLI客户端并连接测试网络](#build-libra-cli-client-and-connect-to-the-testnet).
3.[为Alice和Bob创建账户](#create-alice-s-and-bob-s-account).
4.[铸造并将Libra币增加到Alice和Bob的账户余额](#add-libra-coins-to-alice-s-and-bob-s-accounts).
5.[提交交易](#submit-a-transaction).

##克隆并构建Libra Core

### 克隆Libra Core库

```bash
git clone https://github.com/libra/libra.git
```

### 安装Libra Core

要安装Libra Core，请先切换到 `libra` 目录，并运行相关安装脚本如下：

```
cd libra
./scripts/dev_setup.sh
```
安装脚本执行以下操作：

* 安装rustup &mdash; rustup为Rust语言的安装程序，Libra Core正是基于Rusr实现的；
* 安装指定版本的Rust工具链；
* 安装用于管理编译过程的CMake &mdash;；
* 安装协议缓存编译器protoc &mdash; ；
* 安装用于编译协议缓存的Go &mdash; ；

若安装失败，请参阅[故障排除](#setup)

## 构建Libra CLI客户端并连接测试网络

如下所示，运行客户端，连接到Libra测试网络的validator：

```bash
./scripts/cli/start_cli_testnet.sh
```

该命令利用cargo (Rust的包管理器)构建并运行客户端，并将客户端连接到测试网络上的validator。

一旦连接成功，您将看到如下显示。  若想退出客户端，请使用`quit` 命令

```
usage: <command> <args>

Use the following commands:

account | a
  Account operations
query | q
  Query operations
transfer | transferb | t | tb
  <sender_account_address>|<sender_account_ref_id> <receiver_account_address>|<receiver_account_ref_id> <number_of_coins> [gas_unit_price (default=0)] [max_gas_amount (default 10000)] Suffix 'b' is for blocking.
  Transfer coins from account to another.
help | h
  Prints this help
quit | q!
  Exit this client


Please, input commands:

libra%
```

若此步骤遇到问题，请参阅[故障排除](#client-build-and-run).

<blockquote class="block_note">

**注意**: 若想在您的系统上本地运行validator，可参照[运行本地Validator](#run-a-local-validator-node)。创建账户，铸币和执行交易的操作仍与连接测试网络上的validator相同。

</blockquote>

##为Alice和Bob创建账户

将客户端连接到测试网络后，就可以运行CLI命令创建新账户。  下面，我们一起为两个用户创建账户，我们称他们为Alice和Bob。

### Step 1: 检查CLI客户端是否在系统上运行

当显示**libra%**命令行提示符时，表示您的Libra CLI客户端正在运行。若想查看`account` 的帮助信息，请按如下所示，输入“account” ：

```plaintext
libra% account
usage: account <arg>

Use the following args for this command:

create | c
  Create an account.Returns reference ID to use in other operations
list | la
  Print all accounts that were created or loaded
recover | r <file path>
  Recover Libra wallet from the file path
write | w <file name>
  Save Libra wallet mnemonic recovery seed to disk
mint | mintb | m | mb <receiver account> <number of coins>
  Mint coins to the account.Suffix 'b' is for blocking
```

### Step 2: 创建Alice的账户

需注意，使用CLI创建账户只创建本地密钥对，但不会更新区块链。

输入以下命令行创建Alice的账户：

`libra% account create`

若创建成功，则返回如下：

```plaintext
>> Creating/retrieving next account from wallet
Created/retrieved account #0 address 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8
```

#0 为Alice账户的索引，十六进制的字符串为Alice的账户地址。索引仅仅是引用Alice账户的一种方式。账户索引为本地索引。其他CLI也可以调用，便于用户更方便的引用其所创建的账户。该索引在区块链中无效。只有通过铸币或其他用户的转账，在一笔Libra币被添加到Alice账户中后，Alice的账户才会在区块链上创建。注意：也可在CLI命令行中使用完整的十六进制地址，索引只是引用账户地址的简便方式。

### Step 3: 创建Bob的账户

重复输入前文提到的账户创建命令

`libra% account create`

账户创建成功后，返回如下：

```plaintext
>> Creating/retrieving next account from wallet
Created/retrieved account #1 address 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7
```

#1是Bob账户的索引，十六进制字符串为Bob的账户地址。
更多关于索引详情，请参阅 [创建Alice账户](#step-2-create-alice-s-account)

### Step 4 (可选): 账户列表

输入如下命令行，列出您所创建的账户：

`libra% account list`

成功执行后，返回如下：
```plaintext
User account index: 0, address: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8, sequence number: 0
User account index: 1, address: 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7, sequence number: 0
```
账户的序列号表示该账户发送的交易数。序列号的值随着发起交易的次数等量增加。更多信息请参阅[序号](reference/glossary.md#sequence-number).

##将Libra币增加到Alice和Bob的账户余额

铸造并将Libra币增加到测试网络上Alice和Bob的账户余额的过程，通过Faucet实现。Faucet和测试网络同时运行。该程序仅用于在测试网络上铸币。未来推出的[主网](reference/glossary.md#mainnet)将不会运行Faucet。该程序铸造的Libra币凭空产生，无现实价值。假设您已完成了[为Alice和Bob创建账户](#create-alice-s-and-bob-s-account)，并分别使用了索引0和1。

### Step 1: 添加110个Libra币到Alice账户

输入如下命令，铸造Libra币并将其添加到Alice的账户。

`libra% account mint 0 110`

* 0 为Alice账户索引；
* 110 为添加到Alice账户Libra币的数量。

若成功运行，则会在Libra区块链上创建Alice的账户。

并返回：

```plaintext
>> Minting coins
Mint request submitted
```
请注意，提交交易请求仅意为着该交易被添加到（测试网络validator节点的）内存池中，并不确保该交易的成功执行。稍后，我们将查询账户余额以确认铸币是否成功。

若您账户的铸币命令未能成功提交，请参阅
[故障排除说明](#minting-and-adding-money-to-account)

### Step 2: 添加52个Libra币到Bob账户

输入如下命令，铸造Libra币并将其添加到的bob账户。

`libra% account mint 1 52`

* 1 为Bob账户的索引；
* 52 为添加到Bob账户Libra币的数量。
* 若成功运行，则会在Libra区块链上创建Bob的账户。另一种创建Bob账户的方式为从Alice账户转账给Bob账户。

若成功运行，则返回

```plaintext
>> Minting coins
Mint request submitted
```
若您账户的铸币命令未能成功提交，请参阅
[故障排除说明](#minting-and-adding-money-to-account)

### Step 3: 查看余额

输入如下命令行查看Alice的账户余额：

`libra% query balance 0`

若成功运行，则返回：

`Balance is: 110`

输入如下命令行查看Bob的账户余额：

`libra% query balance 1`

若成功运行，则返回：

`Balance is: 52`

## 提交交易

在提交从Alice账户转账给Bob账户前，需分别查询两个账户的序列号。这将帮助我们更好的理解执行交易时账户的序列号是怎样分别改变的。

### 查询账户序列号

```plaintext
libra% query sequence 0
>> Getting current sequence number
Sequence number is: 0
libra% query sequence 1
>> Getting current sequence number
Sequence number is: 0
```

`query sequence 0`中，0是Alice账户的索引。Alice和Bob账户的序列号均为0，意味着目前为止，并未有Alice或Bob账户发起的交易被执行。

### 转账

输入如下命令行，提交交易请求，将10个Libra币从Alice账户转账到Bob账户：

`libra% transfer 0 1 10`

* 0 is the index of Alice’s account.
* 1 is the index of Bob’s account.
* 10 is the number of Libra to transfer from Alice’s account to Bob’s account.

若成功提交，则返回：

```plaintext
>> Transferring
Transaction submitted to validator
To query for transaction status, run: query txn_acc_seq 0 0 <fetch_events=true|false>
```

您可以使用`query txn_acc_seq 0 0 true` （通过账户和序列号）来查看刚刚提交的交易请求。第一个参数为发起者账户的本地索引，第二个参数是账户的序列好。若该命令行成功运行，则返回如[输出示例](#query-transaction-by-account-and-sequence-number)所示响应。

您刚刚已经提交了交易请求到测试网络validator的节点，该交易会被添加到validator的[内存池](reference/glossary.md#mempool)中。但并不意味着该交易能够成功执行。理论上讲，如果系统运行缓慢或过载，则需要一段时间才能查看结果，您可能需要多次发送查询账户信息的请求。使用 `query account_state 0.` 对索引为0的账户进行查询，若成功，则返回如[输出示例](#query-events)所示响应。

若交易命令遇到任何问题，请参阅 [故障排除说明](#the-transfer-command).

*块交易命令**: 也运用如下所示`transferb` 命令代替`transfer` 命令。`transferb` 命令将提交交易，并在交易被添加到客户端后返回如下响应： 

`libra% transferb 0 1 10`

进一步了解交易从提交、执行到存储的全过程，请参阅[交易生命周期](life-of-a-transaction.md)

###查询交易后的序列号

```plaintext
libra% query sequence 0
>> Getting current sequence number
Sequence number is: 1
libra% query sequence 1
>> Getting current sequence number
Sequence number is: 0
```

Alice账户（索引0）的序列号为1，表示已经有一笔有Alice发起的交易被执行。Bob账户（索引1）的序列号为1，表示目前为止，没有从Bob账户发送的交易被执行。每当账户发送一笔交易，账户的序列号都会加1.

### 查询交易后两个账户的余额

按[此步骤](#step-3-check-the-balance)操作，再次查询两个账户的余额。若交易成功，Alice的账户余额应为100 Libra币，Bob的账户余额应为62 Libra币。

```plaintext
libra% query balance 0
Balance is: 100
libra% query balance 1
Balance is: 62
```

###恭喜！

您已成功在Libra测试网络上进行了第一笔交易，将10 Libra币从Alice账户转给了Bob账户！

## 故障排除说明

### 安装

* 更新Rust
    * 在Libra目录下运行`rustup update` ；
* 更新 protoc:
    * 将`protoc`升级为 3.6.0版本或更高；
* 在Libra目录下，重新运行安装脚本：
    * `./scripts/dev_setup.sh`

### 客户端的构建和运行

若构建客户端失败，请尝试从Libra目录下删除Cargo.lock文件：

* `rm Cargo.lock`

若客户端无法连接到测试网络

* 请检查网络链接；
* 确保正在使用的是最新版本的客户端，拉取最新版本的Libra Core并重新运行客户端：
    * `./scripts/cli/start_cli_testnet.sh`


### 铸币和添加币到账户余额

* 若测试网络中，您所连接的validator节点不可用，您将收到如下所示的“Server unavailable”提示：

  ```plaintext
  libra% account mint 0 110
  >> Minting coins
  [ERROR] Error minting coins: Server unavailable, please retry and/or check **if** host passed to the client is running
  ```
*若交易提交后，余额未更新，请稍等片刻重新查询，区块链上大量交易同时提交可能会导致延迟。  若余额仍未更新，请重新尝试铸币。

* 检查账户是否创建成功，请查询账户状态。查询索引为0的账户，请输入如下命令，

  `libra% query account_state 0`

### 转账命令

若您连接的测试网络validator节点不可用或您与测试网络连接超时，您将收到如下错误提示：

```plaintext
libra% transfer 0 1 10
>> Transferring
[ERROR] Failed to perform transaction: Server unavailable, please retry and/or check if host passed to the client is running
```
解决交易故障：

* 检查与测试网络的连接；
* 查询发起者账户，确定账户存在；对索引为0的用户，输入如下命令：
    * `query account_state 0`
* 使用 `quit` 或 `q!`退出，并重新运行以下命令，以连接到测试网络：
    * `./scripts/cli/start_cli_testnet.sh` from the libra directory

## 查询命令输出示例

### 通过账户和序列号查询交易

此示例为使用账户和序列号查询单个交易的详细信息。

```plaintext
libra% query txn_acc_seq 0 0 true
>> Getting committed transaction by account and sequence number
Committed transaction: SignedTransaction {
 { raw_txn: RawTransaction {
    sender: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8,
    sequence_number: 0,
    payload: {,
      transaction: peer_to_peer_transaction,
      args: [
        {ADDRESS: 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7},
        {U64: 10000000},
      ]
    },
    max_gas_amount: 10000,
    gas_unit_price: 0,
    expiration_time: 1560466424s,
},
 public_key: 55af3fe3f28550a2f1e5ebf073ef193feda44344d94c463b48be202aa0b3255d,
 signature: Signature( R: CompressedEdwardsY: [210, 23, 214, 62, 228, 179, 64, 147, 81, 159, 180, 138, 100, 211, 111, 139, 178, 148, 81, 1, 240, 135, 148, 145, 104, 234, 227, 239, 198, 153, 13, 199], s: Scalar{
  bytes: [203, 76, 105, 49, 64, 130, 162, 81, 22, 237, 159, 26, 80, 181, 111, 94, 84, 6, 152, 126, 181, 192, 62, 103, 130, 94, 246, 174, 139, 214, 3, 15],
} ),
 }
 }
Events:
ContractEvent { access_path: AccessPath { address: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/sent_events_count/" } , index: 0, event_data: AccountEvent { account: 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7, amount: 10000000 } }
ContractEvent { access_path: AccessPath { address: 8337aac709a41fe6be03cad8878a0d4209740b1608f8a81566c9a7d4b95a2ec7, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/received_events_count/" } , index: 0, event_data: AccountEvent { account: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8, amount: 10000000 } }
```

请注意此处交易金额的单位为microlibra。

### 查询交易

以下事例中，我们将查询从索引为0的账户已发送的交易请求。  因为我们只从此账户发送了一次交易请求，所以返回的只有一个交易。  同样返回的还有当前的状态信息，以确保没有交易丢失，交易丢失可能在查询不返回限定的交易时发生。

```plaintext
libra% query event 0 sent 0 true 10
>> Getting events by account and event type.
EventWithProof {
  transaction_version: 3,
  event_index: 0,
  event: ContractEvent { access_path: AccessPath { address: e7460e02058b36d28e8eef03f0834c605d3d6c57455b8ec9c3f0a3c8b89f248b, type: Resource, hash: "217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc97", suffix: "/sent_events_count/" } , index: 0, event_data: AccountEvent { account: 46efbad798a739c088e0e98dd9d592c27c7eb45ba1f8ccbdfc00bd4d7f2947f3, amount: 10000000 } },
  proof: EventProof { ledger_info_to_transaction_info_proof: AccumulatorProof { siblings: [HashValue(62570ae9a994bcb20c03c055667a4966fa50d0f17867dd5819465072fd2c58ba), HashValue(cce2cf325714511e7d04fa5b48babacd5af943198e6c1ac3bdd39c53c87cb84c)] }, transaction_info: TransactionInfo { signed_transaction_hash: HashValue(69bed01473e0a64140d96e46f594bc4b463e88e244b694e962b7e19fde17f30d), state_root_hash: HashValue(5809605d5eed94c73e57f615190c165b11c5e26873012285cc6b131e0817c430), event_root_hash: HashValue(645df3dee8f53a0d018449392b8e9da814d258da7346cf64cd96824f914e68f9), gas_used: 0 }, transaction_info_to_event_proof: AccumulatorProof { siblings: [HashValue(5d0e2ebf0952f0989cb5b38b2a9b52a09e8d804e893cb99bf9fa2c74ab304bb1)] } }
}
Last event state: Some(
    AccountStateWithProof {
        version: 3,
        blob: Some(
            AccountStateBlob {
             Raw: 0x010000002100000001217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc974400000020000000e7460e02058b36d28e8eef03f0834c605d3d6c57455b8ec9c3f0a3c8b89f248b00e1f50500000000000000000000000001000000000000000100000000000000
             Decoded: Ok(
                AccountResource {
                    balance: 100000000,
                    sequence_number: 1,
                    authentication_key: 0xe7460e02058b36d28e8eef03f0834c605d3d6c57455b8ec9c3f0a3c8b89f248b,
                    sent_events_count: 1,
                    received_events_count: 0,
                },
            )
             },
        ),
        proof: AccountStateProof {
            ledger_info_to_transaction_info_proof: AccumulatorProof {
                siblings: [
                    HashValue(62570ae9a994bcb20c03c055667a4966fa50d0f17867dd5819465072fd2c58ba),
                    HashValue(cce2cf325714511e7d04fa5b48babacd5af943198e6c1ac3bdd39c53c87cb84c),
                ],
            },
            transaction_info: TransactionInfo {
                signed_transaction_hash: HashValue(69bed01473e0a64140d96e46f594bc4b463e88e244b694e962b7e19fde17f30d),
                state_root_hash: HashValue(5809605d5eed94c73e57f615190c165b11c5e26873012285cc6b131e0817c430),
                event_root_hash: HashValue(645df3dee8f53a0d018449392b8e9da814d258da7346cf64cd96824f914e68f9),
                gas_used: 0,
            },
            transaction_info_to_account_proof: SparseMerkleProof {
                leaf: Some(
                    (
                        HashValue(c0fbd63b0ae4abfe57c8f24f912f164ba0537741e948a65f00d3fae0f9373981),
                        HashValue(fc45057fd64606c7ca40256b48fbe486660930bfef1a9e941cafcae380c25871),
                    ),
                ),
                siblings: [
                    HashValue(4136803b3ba779bb2c1daae7360f3f839e6fef16ae742590a6698b350a5fc376),
                    HashValue(5350415253455f4d45524b4c455f504c414345484f4c4445525f484153480000),
                    HashValue(a9a6bda22dd6ee78ddd3a42da152b9bd39797b7da738e9d6023f407741810378),
                ],
            },
        },
    },
)
```

###查询账户状态

在以下例子中，我们将查询某一账户的状态。

```plaintext
libra% query account_state 0
>> Getting latest account state
Latest account state is:
 Account: 3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8
 State: Some(
    AccountStateBlob {
     Raw: 0x010000002100000001217da6c6b3e19f1825cfb2676daecce3bf3de03cf26647c78df00b371b25cc9744000000200000003ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a800e1f50500000000000000000000000001000000000000000100000000000000
     Decoded: Ok(
        AccountResource {
            balance: 100000000,
            sequence_number: 1,
            authentication_key: 0x3ed8e5fafae4147b2a105a0be2f81972883441cfaaadf93fc0868e7a0253c4a8,
            sent_events_count: 1,
            received_events_count: 0,
        },
    )
     },
)
 Blockchain Version: 3
```

##运行本地validator节点

要在本地运行validator节点，并创建（未连接到Libra测试网络的）私人区块链网络，请确保您已经正确[安装Libra Core](#setup-libra-core)，即切换到Libra Core的根目录并运行`libra_swarm`，如下所示：

```bash
$ cd ~/libra
$ cargo run -p libra_swarm -- -s
```

`-p libra_swarm` - 使用Cargo运行libra_swarm包，以启动由一个validator组成的本地区块链；

`-S` 启动本地客户端并连接到本地区块链；

若需要查看关于启动validator和连接Libra区块链的更多选项，请运行：

`$ cargo run -p libra_swarm -- -h`

Cargo命令的执行可能需要一些时间来完成。若此命令执行完成，且没有返回错误，则Libra CLI客户端和Libra validator节点已经在您的系统上以实体运行。菜单中将会出现CLI客户端，并返回`libra%` 提示符。

## 交易生命周期

执行完第一笔交易后，您可以参阅[交易生命周期](life-of-a-transaction.md) 以获取关于以下几点的更多信息：

* 从根源上理解交易提交、执行到存储的整个生命周期；
* 了解当Libra生态系统中有交易被提交或执行时，Libra validator的每个逻辑组件之间的交互。

## 参考

* [Welcome page](welcome-to-libra.md).
* [Libra Protocol: Key Concepts](libra-protocol.md) &mdash; Introduces you to the fundamental concepts of the Libra protocol.
* [Getting Started With Move](move-overview.md) &mdash; Introduces you to a new blockchain programming language called Move.
* [Life of a Transaction](life-of-a-transaction.md) &mdash; Provides a look at what happens "under the hood" when a transaction is submitted and executed.
* [Libra Core Overview](libra-core-overview.md) &mdash; Provides the concept and implementation details of the Libra Core components through READMEs.
* [CLI Guide](reference/libra-cli.md) &mdash; Lists the commands (and their usage) of the Libra CLI client.
* [Libra Glossary](reference/glossary.md) &mdash; Provides a quick reference to Libra terminology.
翻译：Jadris Lau 校对：Zhe Wang