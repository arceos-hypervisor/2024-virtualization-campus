# 2023秋冬季训练营第三阶段-虚拟化方向

> 往期视频讲座链接：https://os2edu.cn/course/120

## 前置技能

- 熟悉 x86、arm、risc-v 任一架构
  - 函数调用规范
  - 中断异常处理流程
  - 页表机制实现过程

## 资料阅读建议

- 了解虚拟化相关基础概念
  - 推荐[Docker，K8s，KVM，Hypervisor和微服务有什么区别联系吗？](https://www.zhihu.com/question/307537564/answer/583653317)
- 各体系结构的实现方案介绍
  - [莫策-ARMv8体系结构与硬件虚拟化](./PPT/莫策-ARMv8体系结构与硬件虚拟化.pdf)
  - [齐呈祥-hypercraft设计理念与架构](./PPT/齐呈祥-hypercraft设计理念与架构.pdf)
  - [陈岳-x86虚拟化简述](./PPT/陈岳-x86虚拟化简述.pdf)
- 熟悉 ArceOS
  - 推荐课程：[项目一：ArceOS单内核Unikernel](https://os2edu.cn/course/157) （不做实验，只听石磊老师讲感觉也可
  - [ArceOS 设计&实现 - 组件化OS基础 - 1](https://www.bilibili.com/video/BV1th4y1b7e4/?spm_id_from=333.337.search-card.all.click&vd_source=1d65ea6deb9458981dfc8bd282f7a495)

## Hypervisor讲解

* [x86-64 Hypervisor介绍](./hypervisor/x86_64/README.md)
* [ARM Hypervisor介绍](./hypervisor/aarch64/README.md)
* [RISCV练习](./hypervisor/RISCV/exercises.md)

## 任务列表

* [多核支持](./tasks/multi_core_support.md)
* [多VM支持](./tasks/multi_vm_support.md)
* [内存动态管理](./tasks/dynamic_memory_management.md)
* [设备和中断虚拟化](./tasks/device_and_interrupt_virtualization.md)
* [虚拟机迁移](./tasks/vm_migration.md)
* [实时虚拟机](./tasks/real_time_vm.md)

## 课程PPT

以下ppt对应的课程视频链接：https://os2edu.cn/course/120

* [莫策-ARMv8体系结构与硬件虚拟化](./PPT/莫策-ARMv8体系结构与硬件虚拟化.pdf)
* [莫策-Rust-Shyper代码结构与设计实现](./PPT/莫策-Rust-Shyper代码结构与设计实现.pdf)
* [齐呈祥-hypercraft设计理念与架构](./PPT/齐呈祥-hypercraft设计理念与架构.pdf)
* [李宇-QEMU/KVM基本实现](./PPT/李宇-QEMU-and-KVM基本实现.pdf)
* [李宇-RISC-V-Hypervisor-Extension基本设定](./PPT/李宇-RISC-V-Hypervisor-Extension基本设定.pdf)
* [胡柯洋-Rust-Shyper Monitor VM设计](./PPT/胡柯洋-Rust-Shyper-MonitorVM设计.pdf)
* [胡柯洋-Rust-Shyper多平台兼容和移植经验](./PPT/胡柯洋-Rust-Shyper多平台兼容和移植经验.pdf)
* [胡柯洋-基于Rust的嵌入式虚拟机监视器及热更新技术](./PPT/胡柯洋-基于Rust的嵌入式虚拟机监视器及热更新技术.pdf)
* [季朋-virtio基本原理和驱动/设备交互](https://zhuanlan.zhihu.com/p/639301753?utm_psn=1704906158266068992)
* [陈岳-x86虚拟化简述](./PPT/陈岳-x86虚拟化简述.pdf)
* [陈岳-hcHyper项目架构与实现](./PPT/陈岳-hcHyper项目架构与实现.pdf)

## 参考书籍
* [系统虚拟化原理与实现](./book/系统虚拟化原理与实现.pdf)

## 参考资料
### aarch64
- [Arm® Architecture Reference Manual for A-profile architecture](https://developer.arm.com/documentation/ddi0487/latest/)
- [Arm® Architecture Registers for A-profile architecture](https://developer.arm.com/documentation/ddi0601/latest/)
- [Armv8-A virtualization](https://developer.arm.com/-/media/Arm%20Developer%20Community/PDF/Learn%20the%20Architecture/Armv8-A%20virtualization.pdf?revision=a765a7df-1a00-434d-b241-357bfda2dd31)
- [Arm® Exception model](https://developer.arm.com/-/media/Arm%20Developer%20Community/PDF/Learn%20the%20Architecture/Exception%20model.pdf?revision=a62f2bf2-b08a-4a4f-8cbe-38c67ddf4434)

### x86
- [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)

### RISCV
- [RISC-V Technical Specifications](https://wiki.riscv.org/display/HOME/RISC-V+Technical+Specifications)
- [The RISC-V Instruction Set Manual Volume I: Unprivileged ISA](https://drive.google.com/file/d/1s0lZxUZaa7eV_O0_WsZzaurFLLww7ou5/view)
- [The RISC-V Instruction Set Manual Volume II: Privileged Architecture](https://drive.google.com/file/d/1EMip5dZlnypTk7pt4WWUKmtjUKTOkBqh/view)
- [The RISC-V Hypervisor Extension](https://riscv.org/wp-content/uploads/2017/12/Tue0942-riscv-hypervisor-waterman.pdf)
