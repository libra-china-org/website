---
id: admission-control
title: 准入控制
custom_edit_url: https://github.com/libra/libra/edit/master/admission_control/README.md
---
# 准入控制

准入控制(AC)是Libra的公共API接口，用以接受来自客户端的公共客户端的gRPC请求。

## 概述
准入控制(AC)处理客户端提交的两种类型请求：
1.提交交易 - 向相关validator提交交易请求。
2.请求分布式账本的最新状态- 查询存储，如账户状态，交易日志，验证等。

##实施细节
准入控制(AC)应用于两个API：
1.提交交易(提交交易请求)
    * 对交易请求执行多次验证
	   * 首先验证交易签名若未通过则返回 AdmissionControlStatus::Rejected
	   * 下一步由vm_validator.验证交易。若未通过，则返回相应的VMStatus状态。
	* 当交易通过全部验证，AC将从存储中查询交易发起者的账户余额和最新序列号，并将其和交易请求一起发送的内存池。
    * 若内存池返回MempoolAddTransactionStatus::Valid, 则客户端将接收到AdmissionControlStatus::Accepted  否则，将返回相应的AdmissionControlStatus。
2.请求分布式账本的最新状态AC不执行额外处理。
* 将请求直接发送给存储以进行查阅。

## 模块结构
```
    .
    ├── README.md
    ├── admission_control_proto
    │   └── src
    │       └── proto                           # Protobuf definition files
    └── admission_control_service
        └── src                                 # gRPC service source files
            ├── admission_control_node.rs       # Wrapper to run AC in a separate thread
            ├── admission_control_service.rs    # gRPC service and main logic
            ├── main.rs                         # Main entry to run AC as a binary
            └── unit_tests                      # Tests
```

## AC模块：
和内存池组件交互，以接受客户端提供的交易请求。
和存储组件交互，以查询validator存储
