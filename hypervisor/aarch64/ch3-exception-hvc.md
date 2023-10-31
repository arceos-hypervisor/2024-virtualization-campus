# 3. AARCH64异常处理与HVC指令介绍及其Handler实现
arceos运行在EL1，然而我们需要设置EL2的一些寄存器，所以需要从el1通过hvc陷入el2进行虚拟机相关配置后，再利用eret进入guest执行guest kernel的内容。这部分将介绍aarch64的异常处理与hvc指令，以及这部分的代码实现。
## 3.1 AARCH64异常处理
- 硬件处理部分（发生异常时或主动调用触发异常指令，如svc, hvc, smc）
  - 更新SPSR_ELn，保存PSTATE信息，需要在异常结束返回时恢复该内容
  - 更新PSTATE，反映出当前处理器的状态，比如EL提升
  - 异常返回的地址被存储在ELR_ELn中
  - 跳转到异常向量表执行对应的异常处理函数
  - **注：寄存器的_ELn后缀代表在每个异常等级都有该寄存器的不同拷贝，比如SPSR_EL1和SPSR_EL2时两个不同的物理寄存器**
- 软件处理部分
  - 保存现场（上下文）
  - 根据异常类型执行对应的异常处理函数
  - 恢复现场 (上下文)
  - eret
- 硬件处理部分（执行eret）
  - 把SPSR_ELn恢复到PSTATE中
  - 把PC设置为ELR_ELn，跳转执行
## 3.2 HVC
- 生成一个目标是EL2的异常。
- HVC #\<imm\>：其中bits(16) imm = imm16;
  - 该imm会存于ESR_EL2的imm16（bits [15:0]）。ESR_EL2：用于存储目标EL为EL2发生的异常的相关信息。
## 3.3 相关代码实现
#### 3.3.1 EL2异常向量表（modules/axhal/src/aarch64/trap_el2.S）
```rust
.section .text
.p2align 11
.global exception_vector_base_el2
exception_vector_base_el2:
    // current EL, with SP_EL0
    INVALID_EXCP_EL2 0 0
    INVALID_EXCP_EL2 1 0
    INVALID_EXCP_EL2 2 0
    INVALID_EXCP_EL2 3 0

    // current EL, with SP_ELx
    INVALID_EXCP_EL2 1 1
    HANDLE_IRQ_EL2
    INVALID_EXCP_EL2 2 1
    INVALID_EXCP_EL2 3 1

    // lower EL, aarch64
    HANDLE_LOWER_SYNC
    HANDLE_IRQ_EL2
    INVALID_EXCP_EL2 2 2
    INVALID_EXCP_EL2 3 2

    // lower EL, aarch32
    INVALID_EXCP_EL2 0 3
    INVALID_EXCP_EL2 1 3
    INVALID_EXCP_EL2 2 3
    INVALID_EXCP_EL2 3 3
```
- aarch64的异常向量表一共分为四组，每组中又有四个类别的异常处理，具体如下
  - 四组
    - current el with sp_el0:  没有发生Exception level切换，且SP使用的是SP_EL0
    - current el with sp_elx: 没有发生Exception level切换，且SP使用的是SP_ELx(x=1,2,3)
    - lower el aarch64: 发生Exception level切换，且target level使用的是aarch64
    - lower el aarch32: 发生Exception level切换，且target level使用的是aarch32
  - 四个类别
    - 同步异常（Synchronous Exception）：直接由指令执行触发，返回地址指示了是哪一条指令造成了该异常。如HVC系统调用指令、Data Abort、Instruction Abort等。
    - 异步异常（Asynchronous Exception）
      - IRQ异常：普通中断，优先级低于FIQ。
      - FIQ异常：快速中断，优先级高于IRQ。外部硬件会发出中断请求信号，当当前指令执行完毕时，将引发相应的异常类型。
      - SError异常：System Error。如异步数据异常或部分处理器外部引脚触发等。
##### HANDLE_LOWER_SYNC
- 上述向量表中的INVALID_EXCP_EL2、HANDLE_IRQ_EL2、HANDLE_LOWER_SYNC均为定义在trap_el2.S中的宏。这几个宏的定义流程大致相同，此处详细说明HANDLE_LOWER_SYNC的实现，其余两个类同。
```rust
.macro HANDLE_LOWER_SYNC
.p2align 7
    SAVE_REGS_EL2
    mov     x0, sp
    bl      lower_aarch64_synchronous
    b       .Lexception_return_el2
.endm
```
###### SAVE_REGS_EL2
```rust
.macro SAVE_REGS_EL2
    sub     sp, sp, 34 * 8
    stp     x0, x1, [sp]
    stp     x2, x3, [sp, 2 * 8]
    stp     x4, x5, [sp, 4 * 8]
    stp     x6, x7, [sp, 6 * 8]
    stp     x8, x9, [sp, 8 * 8]
    stp     x10, x11, [sp, 10 * 8]
    stp     x12, x13, [sp, 12 * 8]
    stp     x14, x15, [sp, 14 * 8]
    stp     x16, x17, [sp, 16 * 8]
    stp     x18, x19, [sp, 18 * 8]
    stp     x20, x21, [sp, 20 * 8]
    stp     x22, x23, [sp, 22 * 8]
    stp     x24, x25, [sp, 24 * 8]
    stp     x26, x27, [sp, 26 * 8]
    stp     x28, x29, [sp, 28 * 8]

    mov     x1, sp
    add     x1, x1, #(0x110)
    stp     x30, x1, [sp, 30 * 8]
    mrs     x10, elr_el2
    mrs     x11, spsr_el2
    stp     x10, x11, [sp, 32 * 8]
.endm
```
- 首先调用同样定义在trap_el2.S中的宏SAVE_REGS_EL2。这个宏的作用是把当前异常发生的上下文存储在栈上。如代码所示，将栈指针sp减去存放上下文(即为第二章中说明的Aarch64ContextFrame)的大小，把gpr、sp、elr、spsr存放到栈上。
###### mov x0, sp
- 该语句是将当前sp存放于x0。由于我们之前在栈上已经存储了上下文，所以sp当前指向的刚好是我们保存的上下文内容。此处我们是为了利用x0传递参数，把保存的上下文内容作为参数传给下面的lower_aarch64_synchronous函数.
###### lower_aarch64_synchronous（crates/hypercraft/src/arch/aarch64/exception.rs）
```rust
pub extern "C" fn lower_aarch64_synchronous(ctx: &mut ContextFrame) {
    info!("lower_aarch64_synchronous");
    match exception_class() {
        ...
        0x16 => {
            hvc_handler(ctx);
        }
        ...
    }
}
```
- lower_aarch64_synchronous函数定义于crates/hypercraft/src/arch/aarch64/exception.rs中。可以通过ESR_EL2寄存器获取当前异常的类别，通过类别匹配不同的handler。此处省略了其他异常类别的情况，主要关注hvc_handler。
- hvc_handler（crates/hypercraft/src/arch/aarch64/sync.rs）
  - hvc_handler定义于crates/hypercraft/src/arch/aarch64/sync.rs中。在我们进行hvc调用的时候，会将参数传入x0~x7寄存器，其中x7是作为所有hvc调用的种类和类别，目前只实现了HVC_SYS类别的HVC_SYS_BOOT事件，其中HVC_SYS_BOOT事件包含两个参数，x0为第二阶段翻译的页表基址，x1为对应要运行VM的VCpu的regs。
  - hvc_handler定义于crates/hypercraft/src/arch/aarch64/sync.rs中。在我们进行hvc调用的时候，会将参数传入x0~x7寄存器，其中x7是作为所有hvc调用的种类和类别，目前只实现了HVC_SYS类别的HVC_SYS_BOOT事件，其中HVC_SYS_BOOT事件包含两个参数，x0为第二阶段翻译的页表基址，x1为对应要运行VM的VCpu的regs。
    ```rust
    pub fn hvc_handler(ctx: &mut ContextFrame) {
        let x0 = ctx.gpr(0);
        let x1 = ctx.gpr(1);
        let x2 = ctx.gpr(2);
        let x3 = ctx.gpr(3);
        let x4 = ctx.gpr(4);
        let x5 = ctx.gpr(5);
        let x6 = ctx.gpr(6);
        let mode = ctx.gpr(7);
    
        let hvc_type = (mode >> 8) & 0xff;
        let event = mode & 0xff;
    
        match hvc_guest_handler(hvc_type, event, x0, x1, x2, x3, x4, x5, x6) {
            Ok(val) => {
                ctx.set_gpr(HVC_RETURN_REG, val);
            }
            Err(_) => {
                warn!("Failed to handle hvc request fid 0x{:x} event 0x{:x}", hvc_type, event);
                ctx.set_gpr(HVC_RETURN_REG, usize::MAX);
            }
        }
        if hvc_type==HVC_SYS && event== HVC_SYS_BOOT {
            unsafe {
                let regs: &mut VmCpuRegisters = core::mem::transmute(x1);   // x1 is the vm regs context
                // save arceos context
                regs.save_for_os_context_regs.gpr = ctx.gpr;
                regs.save_for_os_context_regs.sp = ctx.sp;
                regs.save_for_os_context_regs.elr = ctx.elr;
                regs.save_for_os_context_regs.spsr = ctx.spsr;
    
                ctx.gpr = regs.guest_trap_context_regs.gpr;
                ctx.sp = regs.guest_trap_context_regs.sp;
                ctx.elr = regs.guest_trap_context_regs.elr;
                ctx.spsr = regs.guest_trap_context_regs.spsr;
            }
        }
    }
    ```
  - hvc_handler会调用hvc_guest_handler找到对应的类别以及事件执行具体的函数。以当前实现的HVC_SYS_BOOT事件为例，最终会调用实现在crates/hypercraft/src/arch/aarch64/hvc.rs中的init_hv函数。
    - init_hv（crates/hypercraft/src/arch/aarch64/hvc.rs）
    ```rust
    fn init_hv(root_paddr: usize, vm_ctx_addr: usize) {
        // cptr_el2: Condtrols trapping to EL2 for accesses to the CPACR, Trace functionality 
        //           an registers associated with floating-point and Advanced SIMD execution.
        unsafe {
            core::arch::asm!("
                mov x3, xzr           // Trap nothing from EL1 to El2.
                msr cptr_el2, x3"
            );
        }
        msr!(VTTBR_EL2, root_paddr);
        unsafe {
            core::arch::asm!("
                tlbi	alle2         // Flush tlb
                dsb	nsh
                isb"
            );
        }
    
        let regs: &VmCpuRegisters = unsafe{core::mem::transmute(vm_ctx_addr)};
        // set vm system related register
        regs.vm_system_regs.ext_regs_restore();
    }
    ```
    - init_hv函数首先会设置cptr_el2，禁止从el1 trap任何到el2。接着在init_page_table中会设置vttbr_el2为guest page table的root物理地址，而后会刷新TLB。然后通过vm_ctx_addr指针获取到之前设置的vm的相关寄存器，调用VmContext的ext_regs_restore方法把之前设置的和虚拟机相关的系统寄存器真正写入到物理寄存器中。
  - 注意：在if hvc_type == HVC_SYS && event == HVC_SYS_BOOT这个条件语句中，会保存目前arceos触发这个异常时的寄存器，同时会把当前栈上的上下文覆盖为guest trap context。因为当异常处理完毕用eret返回时，我们需要直接跳转到guest kernel entry执行，所以此处需要将原本的上下文修改为之前vcpu初始化的guest上下文。
###### .Lexception_return_el2
```rust
.Lexception_return_el2:
    RESTORE_REGS_EL2
    eret
```
- .Lexception_return_el2首先会调用同样定义在trap_el2.S的RESTORE_REGS_EL2宏。
  - RESTORE_REGS_EL2
  ```rust
  .macro RESTORE_REGS_EL2
    ldp     x10, x11, [sp, 32 * 8]
    msr     elr_el2, x10
    msr     spsr_el2, x11
  
    ldp     x28, x29, [sp, 28 * 8]
    ldp     x26, x27, [sp, 26 * 8]
    ldp     x24, x25, [sp, 24 * 8]
    ldp     x22, x23, [sp, 22 * 8]
    ldp     x20, x21, [sp, 20 * 8]
    ldp     x18, x19, [sp, 18 * 8]
    ldp     x16, x17, [sp, 16 * 8]
    ldp     x14, x15, [sp, 14 * 8]
    ldp     x12, x13, [sp, 12 * 8]
    ldp     x10, x11, [sp, 10 * 8]
    ldp     x8, x9, [sp, 8 * 8]
    ldp     x6, x7, [sp, 6 * 8]
    ldp     x4, x5, [sp, 4 * 8]
    ldp     x2, x3, [sp, 2 * 8]
    ldp     x0, x1, [sp]
    add     sp, sp, 34 * 8
  .endm
  ```
  - RESTORE_REGS_EL2与SAVE_REGS_EL2功能相反，利用ldp指令把全部栈上的上下文的内容重新放回到对应的寄存器中。
- 恢复上下文后，调用eret，返回到触发异常前的EL，跳转到ELR_ELx设定的位置开始执行。
#### 3.3.2 异常向量基址寄存器的设定
- 为了触发异常后能够正确地跳转到异常向量表的地址，需要在之前设定异常向量基址寄存器VBAR_ELx，此处的x指的是异常触发的target level。由于上述异常向量表是为了hypervisor需要处理的一些异常实现的，所以我们需要设定VBAR_EL2。具体代码实现于modules/axhal/src/platform/aarch64_common/boot.rs的_start函数中。
```rust
ldr x8, ={exception_vector_base_el2}    // setup vbar_el2 for hypervisor
msr vbar_el2, x8
```
## 3.4 练习
1. 绘制出在系统中触发hvc异常后程序执行的流程图。（可认为hvc_type为HVC_SYS，hvc_event为HVC_SYS_BOOT）

2. 如果在_start函数中，把异常向量基址寄存器的设定（设定VBAR_EL2）放到“bl {switch_to_el1}”后，会发生什么？为什么会这样？（进阶练习，可选）
     - 提示：可利用make ARCH=aarch64 A=apps/hv HV=y LOG=debug GUEST=nimbos build`进行编译后，利用以下指令进行qemu debug信息输出：
    ```shell
    qemu-system-aarch64 -m 3G -smp 1 -cpu cortex-a72 -machine virt -kernel apps/hv/hv_qemu-virt-aarch64.bin -device loader,file=apps/hv/guest/nimbos/nimbos-aarch64.dtb,addr=0x70000000,force-raw=on -device loader,file=apps/hv/guest/nimbos/nimbos-aarch64.bin,addr=0x70200000,force-raw=on -machine virtualization=on,gic-version=2 -nographic -d int,in_asm -D qemu.log
    ```
    - 此时代码运行会呈现卡住状态，需要用ctrl+a后按x退出。关注log中第一条异常信息，对照[Arm A-profile Architecture Registers](https://developer.arm.com/documentation/ddi0601/latest/)查阅ESR的错误信息，回答问题。

3. 为现有代码添加一个新的hvc_type与hvc_event及其handler，通过x0寄存器作为参数，传递一个数字，在handler中打印出"hello+数字"。并在合适的地方调用这个hvc call作为测试。（挑战练习，可选）