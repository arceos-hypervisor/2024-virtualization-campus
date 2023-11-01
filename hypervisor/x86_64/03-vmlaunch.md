# 3. VM-entry 和 VM-exit

在本阶段，我们将介绍 VM-entry 和 VM-exit 的过程。

> 架构手册：[Intel 64 and IA-32 Architectures Software Developer’s Manual (SDM) Vol. 3C](https://cdrdv2.intel.com/v1/dl/getContent/671447), Chapter 25 ~ Chapter 28

## 3.1 VCPU 上下文切换

在 VM-entry 和 VM-exit 时，我们需要切换 Host 和 Guest 的状态，这一过程即 vCPU 上下文切换 (context switch)，与传统 OS 中线程间、用户内核间的上下文切换类似。

对于保存在 VMCS 中的状态，在 VM-entry 或 VM-exit 时，硬件会自动完成切换。但对于不在 VMCS 中的状态，就需要软件手动进行切换，如下列通用寄存器 (`RSP` 已在 VMCS 状态中)：

| GPRs ||
|-|-|
| `RAX` | `R8 ` |
| `RCX` | `R9 ` |
| `RDX` | `R10` |
| `RBX` | `R11` |
| ~~RSP~~ | `R12` |
| `RBP` | `R13` |
| `RSI` | `R14` |
| `RDI` | `R15` |

因此，我们需要在 VM-entry 时，先手动从内存载入 Guest 通用寄存器，再执行 `VMLAUNCH`/`VMRESUME`，让硬件自动切换 VMCS 中的状态，然后进入 Guest。在 VM-exit 时，硬件先自动完成了 VMCS 中状态的切换，但此时的通用寄存器还是 Guest 的且没有被保存，需要将它们保存到内存中，以便处理 VM-exit 时能够访问。

由于在我们的实现中，通过 `VMLAUNCH` 进入 Guest 后，不再执行之后的指令。当发生 VM-exit 回到 Host 模式时，也是跳转到 VM-exit 处理函数，并不会回到 `VMLAUNCH` 之后。VM-exit 时也无需用到之前保存的 Host 通用寄存器。因此，我们可以无需保存与恢复每次 VM-entry 前的 Host 通用寄存器。

## 3.2 处理 VM-exit

### 3.2.1 获取 VM-exit 信息

首先通过 VMCS VM-exit information fields 中的 **Exit reason** 字段，获取 VM-exit 的基本信息，其中的主要位有：

* Basic exit reason：基本原因，见 SDM Vol. 3D, Appendix C。
* VM-entry failure：表明 VM-entry 失败，如发生了原因为 "VM-entry failure due to invalid Guest state" 或 "VM-entry failure due to MSR loading" 的 VM-exit。

其他常用的 VMCS VM-exit information fields 有：

| VMCS 字段 | 描述 |
|-|-|
| Exit qualification | 因 I/O 指令、EPT violation、控制寄存器访问等导致的 VM-exit 的详细信息 |
| VM-exit instruction length | 因执行特定指令导致 VM-exit 时的指令长度 |
| VM-instruction error field | VMX 指令执行失败时错误码 |
| Guest-physical address | EPT violation 时的出错 Guest 物理地址 (Step 4) |
| VM-exit interruption information | 因外部中断导致 VM-exit 时的详细信息 (Step 6)  |

## 3.3 实现

本节的内容位于`crates/hypercraft/src/arch/x86_64/vmx/vcpu.rs`中。首先是 `run` 函数：

```rust
/// Run the guest, never return.
pub fn run(&mut self) -> ! {
    VmcsHostNW::RSP
        .write(&self.host_stack_top as *const _ as usize)
        .unwrap();
    unsafe { self.vmx_launch() }
}
```

该函数首先将 VMCS Host `RSP` 设为 `self.host_stack_top` 的地址，方便 VM-exit 时保存 Guest 寄存器。之后进入 `vmx_launch` 函数：

```Rust
 #[naked]
unsafe extern "C" fn vmx_launch(&mut self) -> ! {
    asm!(
        "mov    [rdi + {host_stack_top}], rsp", // save current RSP to Vcpu::host_stack_top
        "mov    rsp, rdi",                      // set RSP to guest regs area
        restore_regs_from_stack!(),
        "vmlaunch",
        "jmp    {failed}",
        host_stack_top = const size_of::<GeneralRegisters>(),
        failed = sym Self::vmx_entry_failed,
        options(noreturn),
    )
}
```

这段代码是用纯汇编写成的，流程如下：

1. 将当前的 Host `RSP` 保存进 `self.host_stack_top` (`RDI` 是该函数的第一个参数，即 `self`)。
2. 切换 Host `RSP` 到 `RDI`，即 `self.guest_regs` 结构的开头。
3. 通过 `restore_regs_from_stack!()` 宏将 Guest 的通用寄存器恢复出来。
4. 执行 `VMLAUNCH`，进入 Guest。
5. 如果 `VMLAUNCH` 执行失败，将不会进入 Guest，而是执行之后的指令，这里就直通跳转到了错误处理函数 `vmx_entry_failed` (该函数会直接 panic)。

![VCPU 上下文切换](figures/3-context-switch.svg)

VMCS 中将 Host `RIP` 设置为 `vmx_exit` 函数的地址，因此 VM-exit 时会直接跳转到该函数：

```Rust
#[naked]
unsafe extern "C" fn vmx_exit(&mut self) -> ! {
    asm!(
        save_regs_to_stack!(),
        "mov    r15, rsp",                      // save temporary RSP to r15
        "mov    rdi, rsp",                      // set the first arg to &Vcpu
        "mov    rsp, [rsp + {host_stack_top}]", // set RSP to Vcpu::host_stack_top
        "call   {vmexit_handler}",              // call vmexit_handler
        "mov    rsp, r15",                      // load temporary RSP from r15
        restore_regs_from_stack!(),
        "vmresume",
        "jmp    {failed}",
        host_stack_top = const size_of::<GeneralRegisters>(),
        vmexit_handler = sym Self::vmexit_handler,
        failed = sym Self::vmx_entry_failed,
        options(noreturn),
    );
}
```

其流程如下：

1. 通过 `save_regs_to_stack!()` 宏将 Guest 的通用寄存器保存到 `guest_regs` 中。
2. 保存此时的 Host `RSP` 到临时寄存器 `R15`，便于之后再次 VM-entry 时恢复 Guest 通用寄存器。
3. 将 `RDI` 设为 `RSP`，即接下来调用的 `vmexit_handler` 函数的第一个参数为当前 `VmxVcpu` 结构。
4. 从 `VmxVcpu::host_stack_top` 恢复 Host `RSP`。
5. 调用 VM-exit 处理函数 `vmexit_handler`。
6. VM-exit 处理完毕，从 `R15` 恢复栈，并恢复 Guest 通用寄存器，准备再次进入 Guest。
7. 执行 `VMRESUME`，再次进入 Guest。
8. 如果 `VMRESUME` 执行失败，直通跳转到 `VmxVcpu::vmx_entry_failed()`。

## 3.4 练习

1. 阅读代码，详细阐述：
    1. 在 VM-entry 和 VM-exit 的过程中，Host 的 `RSP` 寄存器的值是如何变化的？包括：哪些指令
    2. 在 VM-entry 和 VM-exit 的过程中，Guest 的通用寄存器的值是如何保存又如何恢复的？（提示：与`RSP`的修改有关）
    3. VM-exit 过程中是如何确保调用 `vmexit_handler` 函数时栈是可用的？
