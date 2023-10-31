# Arceos x86-64 Hypervisor介绍

本文档介绍x86-64下基于arceos的Hypervisor的设计和实现。此Hypervisor的设计和实现源自[RVM-Tutorial](https://github.com/rcore-os/RVM-Tutorial.git)，文档中大部分也来自于[RVM-Tutorial的文档](https://github.com/equation314/RVM-Tutorial/wiki)。

## 目录

* [0. 开发环境和代码结构](./00-env.md)
* [1. Intel VMX简介和初始化](./01-vmx.md)
* [2. VMCS 配置](./02-vmcs.md)
* [3. VM-entry 和 VM-exit](./03-vmlaunch.md)
* [4. EPT 与内存隔离](./04-ept.md)
* [5. 运行一个相对完整的Guest OS](./05-nimbos.md)
* [6. I/O 和中断虚拟化](./06-io-int.md)
