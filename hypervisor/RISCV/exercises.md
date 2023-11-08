
## 练习1
1. 在arceos中运行linux
2. （1）阅读[The RISC-V Instruction Set Manual Volume II: Privileged Architecture](https://drive.google.com/file/d/1EMip5dZlnypTk7pt4WWUKmtjUKTOkBqh/view) Chapter 8(8.1)Privilege Modes，回答RISCV引入虚拟化扩展后，其特权模式有哪些新的变化，各模式之间存在什么关系与区别。（2）请问Hypercraft是哪一种hypervisor。

## 练习2
1. 在合适的地方修改代码，打印出在vcpu初始化前后VmCpuRegisters各个部分寄存器的值。比较它们哪些值发生了变化。并根据本章节介绍的内容，对照[The RISC-V Instruction Set Manual Volume II: Privileged Architecture](https://drive.google.com/file/d/1EMip5dZlnypTk7pt4WWUKmtjUKTOkBqh/view)，说明值变化的寄存器的作用。

## 练习3
1. 参考[The RISC-V Instruction Set Manual Volume II: Privileged Architecture](https://drive.google.com/file/d/1EMip5dZlnypTk7pt4WWUKmtjUKTOkBqh/view) Chapter 8(8.6) Traps，分析RISCV版本的Hypercraft存在哪些异常类型，这些异常从触发到处理完成分别经过怎样的特权级别的切换，会涉及到哪些寄存器的设置。

## 练习4
1. 假设在 guest OS 中启用了 3 级页表，如果 guest 的一次访存 (使用 guest 虚拟地址) 发生了 TLB 缺失，请问以riscv64 two-stage address translation的方法实现内存虚拟化时，最多会导致多少次内存访问？

## 练习5
1. 把kernel和dtb放到其他的内存地址后启动。
    提示：除了修改apps/hv/src/main.rs中载入kernel和dtb的地址，还有哪些地方需要修改？关注scripts/make/qemu.mk、apps/hv/guest/linux/linux.dts，注意dts如何编译成dtb。
