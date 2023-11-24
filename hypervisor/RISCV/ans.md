# RISCV练习答案
## 练习1：
1.略
2.
（1）引入虚拟化扩展后，增加了HS模式（是S模式的一种扩展），Hypervisor位于该模式，可以托管客户机操作系统。VS模式和VU模式，VS模式是客户机内核所处的模式，VU是客户机用户程序处于的模式。具体的可以看下图：
![特权模式图](./img/privilege_modes.png)

（2）二型虚拟机，因为是运行于arceos中，并不是直接运行在裸机上。

## 练习2：
主要涉及以下寄存器：
- hstatus 寄存器（Hypervisor Status）：是HS模式下用于控制和存储的一些状态信息的寄存器，用于控制中断使能、虚拟内存设置以及一些其他的特权级别控制。
- sstatus寄存器（Supervisor Status）：用于控制和存储Supervisor模式下的状态信息。包括位字段，用于控制中断使能、虚拟内存设置等。
- sepc寄存器（Supervisor Exception Program Counter）：存储导致最近一次异常或中断的指令地址。
- hgatp寄存器（Hypervisor Guest Address Translation and Protection）：用于控制和配置HS模式下的虚拟地址转换和保护机制。

## 练习3：
在Hypercraft中产生Trap主要的起因是Interrupt和Exception，Interrupt和Exception所包含的具体类型定义如下面代码显示：

```rust
///Trap
pub enum Trap {
    Interrupt(Interrupt),
    Exception(Exception),
}

/// Interrupt
pub enum Interrupt {
    UserSoft,
    VirtualSupervisorSoft,
    SupervisorSoft,
    UserTimer,
    VirtualSupervisorTimer,
    SupervisorTimer,
    UserExternal,
    VirtualSupervisorExternal,
    SupervisorExternal,
    Unknown,
}

/// Exception
pub enum Exception {
    InstructionMisaligned,
    InstructionFault,
    IllegalInstruction,
    Breakpoint,
    LoadFault,
    StoreMisaligned,
    StoreFault,
    UserEnvCall,
    VirtualSupervisorEnvCall,
    InstructionPageFault,
    LoadPageFault,
    StorePageFault,
    InstructionGuestPageFault,
    LoadGuestPageFault,
    VirtualInstruction,
    StoreGuestPageFault,
    Unknown,
}
```

可能存在如下特权级别切换：客户机用户程序（VU模式）—>客户机内核（VS模式）—>Hypervisor（HS模式）

主要涉及下面一些寄存器：
- scause：S模式下保存产生异常的原因
- stval：S模式下存储导致异常的指令或数据的值
- htval：HS模式下记录异常导致的指令或数据值
- htinst：HS模式下储导致异常的指令的副本


## 练习4：
guest OS 中启用了 3 级页，在gva->gpa的过程中，由于还涉及guest页表gpa->hpa的转换，所以需要处理3×3=9次。最后得到了gpa，gpa->hpa还需要进行第二阶段的3次访存，所以是9+3=12次。

## 练习5：
分别修改apps/hv/src/main.rs处理dtb与kerel entry point的相关内容、apps/hv/guest/inux/linux.dts中memory节点的reg并利用dtc重新编译为dtb、以及srcipts/make/gemu.mk 51、52行内容。
