# 0. 环境准备
## 0.1 开发环境
- 硬件：
  - 具有 Intel CPU 的计算机
  - 支持 VT-x (一般都支持)，并在 BIOS 中已起用
- 软件：
  - 建议安装 Ubuntu 等 Linux 操作系统 (不能是虚拟机)
  - 已启用 KVM
  > 由于我们要在 QEMU 里运行我们的 hypervisor，相当于在虚拟机里运行虚拟机，所以需要用 KVM 提供嵌套虚拟化支持。
  - QEMU >= 7.0.0
  - Rust 工具链 (nightly-2023-03-03)