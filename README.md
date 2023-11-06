# 2023秋冬季训练营第三阶段-虚拟化方向

## Hypervisor讲解

* [x86-64 Hypervisor介绍](./hypervisor/x86_64/README.md)
* [ARM Hypervisor介绍](./hypervisor/aarch64/README.md)

## 任务列表

* [多核支持](./tasks/multi_core_support.md)
* [多VM支持](./tasks/multi_vm_support.md)
* [内存动态管理](./tasks/dynamic_memory_management.md)
* [设备和中断虚拟化](./tasks/device_and_interrupt_virtualization.md)
* [虚拟机迁移](./tasks/vm_migration.md)
* [实时虚拟机](./tasks/real_time_vm.md)


## 课程PPT

* [莫策-ARMv8体系结构与硬件虚拟化](./PPT/莫策-ARMv8体系结构与硬件虚拟化.pdf)
* [莫策-Rust-Shyper代码结构与设计实现](./PPT/莫策-Rust-Shyper代码结构与设计实现.pdf)
* [齐呈祥-hypercraft设计理念与架构](./PPT/齐呈祥-hypercraft设计理念与架构.pdf)
* [李宇-QEMU/KVM基本实现](./PPT/李宇-QEMU/KVM基本实现.pdf)
* [李宇-RISC-V-Hypervisor-Extension基本设定](./PPT/李宇-RISC-V-Hypervisor-Extension基本设定.pdf)
* [](./PPT/.pdf)

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
