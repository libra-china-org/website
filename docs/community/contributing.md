---
id: contributing
title: 贡献代码
---

我们致力于简单化透明化Libra项目的贡献过程

<blockquote class="block_note">
Libra Core项目目前仍为早期原型，并正经历快速发展。在做出任何贡献前，请先参与论坛讨论，确保它符合项目的路线图。
</blockquote>

## 成为Libra Core的贡献者

成为Libra Core的贡献者需要您确保安装了代码库的最新版本。运行以下命令行，以完整安装Libra Core，并用以分析、测试和构建文档
```
$ git clone https://github.com/libra/libra.git
$ cd libra
$ ./scripts/dev_setup.sh
$ source ~/.cargo/env
$ cargo build
$ cargo test
```

##编程指南

更详细的贡献方式请参阅[编程指南](coding-guidelines.md).

## 文档

所有的开发者文档都会被公布在在Libra开发者站点中。开发者站点是开源的，其构建代码请参阅 [资源库](https://github.com/libra/website/).开发者站点使用[Docusaurus](https://docusaurus.io/)构建。

若您熟悉Markdowm的使用，现在就成为开发者的一员吧！

## Pull Requests

在早期发展阶段，我们计划仅查看并评估pull request。当代码库稳定后，我们就能开始接受社区中提交的pull request。

提交pull request，你需要：

1.通过fork`libra` ，在`master`中建立分支。
2.若你添加了需要测试的代码，请一并提交测试单元。
3.若你对API有所改动, 请更新相关文档，创建并测试开发者站点。
4.验证并确认其通过检验组件。
5.确保你的代码通过两个指针。
6.若没有签署贡献者许可协议，请先同意协议。
7.提交pull request。

## 贡献者许可协议

您必须先签署贡献者许可协议，Libra项目才会接受你的pull request。您只需签署一次协议，不论您为什么Libra项目，多少项目工作。仅代表自己的独立贡献者的可以签署[个人CLA](https://github.com/libra/libra/blob/master/contributing/individual-cla.pdf).代表公司的贡献者需请求上级签署[企业CLA](https://github.com/libra/libra/blob/master/contributing/corporate-cla.pdf).

## 行为守则
请参阅[行为守则](../policies/code-of-conduct.md)，其规范了对社区内交流的期望。

## 问题

Libra使用[GitHub issues](https://github.com/libra/libra/issues)跟踪bug。请提交能够重现您所面对问题的必要信息和相关指引。安全性相关bug请使用[security procedures](../policies/security.md)提交。

翻译：Jadris Lau 校对：Zhe Wang