
## 练习1
1. 在arceos中运行nimbos
2. 阅读[Armv8-A Virtualization](https://developer.arm.com/-/media/Arm%20Developer%20Community/PDF/Learn%20the%20Architecture/Armv8-A%20virtualization.pdf?revision=a765a7df-1a00-434d-b241-357bfda2dd31) Chapter 2（共4页），回答hypercraft是哪一种hypervisor。

## 练习2
1. 在合适的地方修改代码，打印出在vcpu初始化前后regs中guest_trap_context_regs与vm_system_regs各个寄存器的值。比较它们哪些值发生了变化。并根据本章节介绍的内容，对照[Arm A-profile Architecture Registers](https://developer.arm.com/documentation/ddi0601/latest/)，说明值变化的寄存器的作用。

## 练习3
1. 绘制出在系统中触发hvc异常后程序执行的流程图。（可认为hvc_type为HVC_SYS，hvc_event为HVC_SYS_BOOT）

2. 如果在_start函数中，把异常向量基址寄存器的设定（设定VBAR_EL2）放到“bl {switch_to_el1}”后，会发生什么？为什么会这样？（进阶练习，可选）
     - 提示：可利用make ARCH=aarch64 A=apps/hv HV=y LOG=debug GUEST=nimbos build`进行编译后，利用以下指令进行qemu debug信息输出：
    ```shell
    qemu-system-aarch64 -m 3G -smp 1 -cpu cortex-a72 -machine virt -kernel apps/hv/hv_qemu-virt-aarch64.bin -device loader,file=apps/hv/guest/nimbos/nimbos-aarch64.dtb,addr=0x70000000,force-raw=on -device loader,file=apps/hv/guest/nimbos/nimbos-aarch64.bin,addr=0x70200000,force-raw=on -machine virtualization=on,gic-version=2 -nographic -d int,in_asm -D qemu.log
    ```
    - 此时代码运行会呈现卡住状态，需要用ctrl+a后按x退出。关注log中第一条异常信息，对照[Arm A-profile Architecture Registers](https://developer.arm.com/documentation/ddi0601/latest/)查阅ESR的错误信息，回答问题。

3. 为现有代码添加一个新的hvc_type与hvc_event及其handler，通过x0寄存器作为参数，传递一个数字，在handler中打印出"hello+数字"。并在合适的地方调用这个hvc call作为测试。（挑战练习，可选）

## 练习4
1. 假设在 guest OS 中启用了 3 级页表，如果 guest 的一次访存 (使用 guest 虚拟地址) 发生了 TLB 缺失，请问以aarch64 stage2 translation的方法实现内存虚拟化时，最多会导致多少次内存访问？(均为 3 级页表)
2. 在apps/hv/src/main.rs的set_gpm函数中，打印建立地址映射前后查询ipa的结果。（进阶练习，可选）

## 练习5
1. 把kernel和dtb放到其他的内存地址后启动。可以把它们放到0x40000000吗？为什么？
    提示：除了修改apps/hv/src/main.rs中载入kernel和dtb的地址，还有哪些地方需要修改？关注scripts/make/qemu.mk、apps/hv/guest/nimbos/nimbos-aarch64.dts，注意dts如何编译成dtb。
2. 分析如果需要启动linux，还需要哪些虚拟化功能的支持。（挑战练习，可选）
