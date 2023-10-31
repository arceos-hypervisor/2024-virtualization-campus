# 0. 开发环境和代码结构

## 0.1 开发环境

* 硬件：
    * 具有 Intel CPU 的计算机
    * CPU 支持 VT-x（现代 Intel CPU 一般都支持），并在 BIOS 中已启用
* 软件：
    * 基于 Linux 内核的操作系统，如 Ubuntu 等
        * 推荐直接安装在物理机上，如果使用虚拟机则需要正确配置嵌套虚拟化
    * [启用 KVM](https://www.linux-kvm.org/page/Choose_the_right_kvm_%26_kernel_version)
    * [QEMU](https://www.qemu.org/download/) >= 7.0.0
    * [Rust 工具链](https://www.rust-lang.org)（nightly channel）

## 0.2 代码组织结构

这里仅仅介绍 `arceos` 仓库中直接与 Hypervisor 直接相关的部分。

* `crates/hypercraft/`: Hypercraft 的代码。Hypercraft 是基于 arceos 实现的一个 Type-1 Hypervisor 库。
* `app/hv/`: 使用 Hypercraft 实现的 Hypervisor。
* `modules/axruntime/src/hv/`: 运行时 arceos 给 Hypervisor 提供服务的代码，包括内存的分配和管理，虚拟设备的实现。
* `crates/page_table_entry/src/arch/x86_64/`: EPT 的定义。