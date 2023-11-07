
## 练习1
1. 在arceos中运行linux
2. 阅读[RISCV Hypervisor Extension, Version 1.0](https://five-embeddev.com/riscv-isa-manual/latest/hypervisor.html#)，重点阅读[5.1 Privilege Modes](https://five-embeddev.com/riscv-isa-manual/latest/hypervisor.html#privilege-modes)、[5.2 Hypervisor and Virtual Supervisor CSRs](https://five-embeddev.com/riscv-isa-manual/latest/hypervisor.html#hypervisor-and-virtual-supervisor-csrs)与[5.6 Traps](https://five-embeddev.com/riscv-isa-manual/latest/hypervisor.html#traps)，结合[齐呈祥-hypercraft 的实现](https://os2edu.cn/course/120/replay/5796)了解riscv64的虚拟化。

## 练习2
1. 在合适的地方修改代码，打印出在vcpu初始化前后regs中guest_regs各个寄存器的值。比较它们哪些值发生了变化。并根据本章节介绍的内容，对照[RISCV Hypervisor Extension, Version 1.0](https://five-embeddev.com/riscv-isa-manual/latest/hypervisor.html#)，说明值变化的寄存器的作用。

## 练习3
1. 假设在 guest OS 中启用了 3 级页表，如果 guest 的一次访存 (使用 guest 虚拟地址) 发生了 TLB 缺失，请问以riscv64 two-stage address translation的方法实现内存虚拟化时，最多会导致多少次内存访问？(均为 3 级页表)
2. 在apps/hv/src/main.rs的set_gpm函数中，打印建立地址映射前后查询ipa的结果。（进阶练习，可选）

## 练习4
1. 把kernel和dtb放到其他的内存地址后启动。
    提示：除了修改apps/hv/src/main.rs中载入kernel和dtb的地址，还有哪些地方需要修改？关注scripts/make/qemu.mk、apps/hv/guest/linux/linux.dts，注意dts如何编译成dtb。
