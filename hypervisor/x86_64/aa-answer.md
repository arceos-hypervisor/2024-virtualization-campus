# 各节练习参考答案

## 1.6

1. 略
2. `hardware_enable` 函数调用 `VmxRegion::new` 以分配并初始化 `vmx_region`。`VmxRegion::new` 调用 `PhysFrame::alloc_zero` 以分配一页4KB的物理内存并清零，然后在此页的前4字节写入 `revision_id`。

## 2.6

1. 如下：
    1. 如下：
        1. 设置 `Primary Processor-Based VM-Execution Controls` 字段中的 `HLT exiting` 位；
        2. 设置 `Primary Processor-Based VM-Execution Controls` 字段中的 `PAUSE exiting` 位；
        3. 设置 `Pin-Based VM-Execution Controls` 中的 `External-interrupt exiting` 位；
        4. Guest 的 `rflags` 寄存器存放在 `VMCS` 的 `Guest-state area` 中，使用 `VMREAD/VMWRITE` 即可编辑；
        5. 设置 `Exception Bitmap` 的 14 位（#PF的编号），同时要正确设置 `Page-Fault Error-Code Mask` 和 `Page-Fault Error-Code Match`；
        6. 设置 `Primary Processor-Based VM-Execution Controls` 中的 `Unconditional I/O exiting` 位；
        7. 设置 `Primary Processor-Based VM-Execution Controls` 中的 `Use I/O bitmaps` 位，分配两个4KB页存储 `I/O bitmaps A` 和 `I/O bitmaps B`，设置其中对应 `0x3f8` 的位，并在 `VMCS` 中设置其地址；
        8. 清空 `Primary Processor-Based VM-Execution Controls` 中的 `Use MSR bitmaps` 位；
        9. 设置 `Primary Processor-Based VM-Execution Controls` 中的 `Use MSR bitmaps` 位，分配一个4KB页存储 `MSR bitmap`，设置其中控制 `IA32_EFER(0xC0000080)` 写入的位，并在 `VMCS` 中设置其地址；
        10. 设置 `Cr0 Guest/Host Masks` 的第 31 位。
    2. 首先要给每一个 vCPU 创建并配置一个 `VMCS`。在切换 vCPU 时，要把目标 vCPU 的 `VMCS` 设置为“当前 `VMCS`”，并切换其他不在 `VMCS` 中的 Guest 状态。

## 3.4

1. 如下：
    1. RSP的变化过程：
        1. 在 VM-entry 的过程中，RSP 的原始值被保存到 `VmxVcpu` 结构体的 `host_stack_top` 字段中，然后 RSP 被设置为 `VmxVcpu` 结构体的地址，让 `restore_regs_from_stack` 宏能从 `VmxVcpu` 结构体的 `guest_regs` 字段中弹出 Guest 通用寄存器的值。执行 `vmlaunch` 时，RSP 被切换为 `VMCS Guest-state area` 中记录的 Guest RSP。
        2. 在 VM-exit 时，Guest 的 RSP 被保存到 `VMCS Guest-state area` 中，CPU 从 `VMCS Host-state area` 中恢复 Host RSP（被提前设置为 `VmxVcpu` 结构体的 `host_stack_top` 字段的地址），然后 `save_regs_to_stack` 将 Guest 通用寄存器的值压入 `VmxVcpu` 结构体的 `guest_regs` 字段中；Guest 通用寄存器保存完成后，RSP 指向 `VmxVcpu` 结构体的地址，代码一方面把这个地址保存到 R15 寄存器中以供后面使用（因为 R15 是被调用者保存的寄存器，在调用 `vmexit_handler` 后保持不变），另一方面保存到 RDI 寄存器中，作为调用 `vmexit_handler` 的参数；然后，代码从 `VmxVcpu` 结构体的 `host_stack_top` 字段中恢复VM-entry前的 RSP 并调用 `vmexit_handler`；调用完成后，按照和 VM-entry 中一致的程序恢复 Guest 通用寄存器的值并回到 Guest 中。
    2. Guest 通用寄存器的值保存在 `VmxVcpu` 结构体的 `guest_regs` 字段中，由 `restore_regs_from_stack` 和 `save_regs_to_stack` 两个宏负责恢复和保存。具体来说，恢复和保存时，将 RSP 临时指向 `VmxVcpu` 结构体的 `guest_regs` 字段，然后用 push/pop 指令读写其中的值到通用寄存器。
    3. 在 VM-entry 时，代码保存了 RSP 的值到 `VmxVcpu` 结构体的 `host_stack_top` 字段中；在 VM-exit 时调用 `vmexit_handler` 前，代码会将栈切回这里，从而给 `vmexit_handler` 提供一个可用的栈。

## 4.4

1. 考虑每个 Guest VM 的操作系统也需要一份单独的页表，共需要 44 份影子页表；
2. 开启 4 级页表后，Guest 的一次访存实际需要访问 5 次 Guest 内存，而每次访问 Guest 内存又需要 5 次物理内存访问，故一共 25 次；
3. 主要需要处理 EPT Violation，如果并非真的非法访问而是访问尚未分配的内存，则动态分配一定量的内存给Guest。为此还需要实现合适的分配和记录机制。

## 5.4

1. prot_gdt 是一个有两个有效项的全局描述符表，其中定义了一个代码段和一个数据段，均为4GB大小的对等映射。prot_gdt_desc 是指向这个全局描述符表的描述符，记录了其地址和大小，用作 lgdt 指令的参数；
2. 修改 `GUEST_PHYS_MEMORY_SIZE` 为 `0x200_0000`；
3. 修改 `GUEST_ENTRY` 以让 NimbOS 内核被复制到其他地址；同时修改 NimbOS BIOS 中的跳转地址。

## 6.1

1. 编写一个结构体并正确实现 `PortIoDevice` trait，然后创建一个该结构体添加到 `VIRT_DEVICES` 中；
2. 为了让虚拟设备能保存并修改内部状态，需要修改 `PortIoDevice` trait，在其 `read` 和 `write` 函数中使用 `&mut self`。
