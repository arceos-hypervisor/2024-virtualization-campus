# 0. 环境准备
## 0.1 开发环境
- 软件：
  - 建议安装 Ubuntu 等 Linux 操作系统 (不能是虚拟机)
  - [QEMU](https://www.qemu.org/download/) >= 7.0.0
  - [Rust 工具链](https://www.rust-lang.org)（nightly-2023-03-03）
    - 理论上只需指定 nightly channel 即可，但考虑到不同的 nightly 编译器可能出现行为差异，请在遇见奇怪的问题时尝试锁定工具链版本
