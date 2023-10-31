# 2. CPU、VCpu与VM结构
## 2.1 CPU （crates/hypercraft/src/arch/aarch64/cpu.rs）
该结构对应于物理CPU，用于初始化物理CPU虚拟化所需内容，将其保存在栈上，并保存保存对应的VCpu，该部分主要用于后续多CPU体系的实现。
```rust
pub struct PerCpu<H:HyperCraftHal>{ 
    /// per cpu id
    pub cpu_id: usize,
    stack_top_addr: HostVirtAddr,
    /// save for correspond vcpus
    pub vcpu_queue: Mutex<VecDeque<usize>>,
    marker: core::marker::PhantomData<H>,
}
```
### 2.1.1 成员变量
- cpu_id： 当前物理CPU的id
- stack_top_addr：存储该物理CPU的boot stack地址
- vcpu_queue：存储当前物理CPU对应的VCpu
### 2.1.2 方法
- pub fn init(boot_id: usize, stack_size: usize) -> HyperResult\<()\>
  - 为每个CPU分配内存页用于存储CPU的结构信息，并将当前运行的CPU id设置为boot CPU
- pub fn setup_this_cpu(cpu_id: usize) -> HyperResult<()>
  - 将cpu_id对应的CPU存储在TPIDR_EL1中，作为当前运行的CPU
- pub fn this_cpu() -> &'static mut PerCpu\<H\>
  - 通过读取TPIDR_EL1寄存器，得到当前运行的CPU结构体
- pub fn create_vcpu(&mut self, vcpu_id: usize) -> HyperResult<VCpu\<H\>>
  - 为当前CPU创建一个VCpu，加入到vcpu_queue中
> TPIDR_EL1：EL1 Software Thread ID Register。提供一个用于存储线程标识信息的位置，以供在EL1特权级执行的软件进行操作系统管理。
## 2.2 VCpu（crates/hypercraft/src/arch/aarch64/vcpu.rs）
该结构对应于虚拟机运行的虚拟CPU，用于存储进入guest之前或退出guest时guest的Trap Context以及客户机系统寄存器的值，同时存储进入guest时host的Trap Context值。
```rust
pub struct VCpu<H:HyperCraftHal> {
    /// Vcpu id
    pub vcpu_id: usize,
    /// Vcpu context
    pub regs: VmCpuRegisters,
    marker: PhantomData<H>,
}
```
### 2.2.1 成员变量
- vcpu_id: vcpu对应的id
- regs：存储的相关寄存器状态
    ```rust
    pub struct VmCpuRegisters {
        /// guest trap context
        pub guest_trap_context_regs: ContextFrame,
        /// arceos context
        pub save_for_os_context_regs: ContextFrame,
        /// virtual machine system regs setting
        pub vm_system_regs: VmContext,
    }
    ```
  - guest_trap_context_regs & save_for_os_context_regs
    - guest_trap_context_regs存储了进入guest之前或退出guest时guest的trap context，save_for_os_context_regs存储了进入guest之前arceos本身的trap context。
    - trap context具体内容见下，Aarch64ContextFrame实现于crates/hypercraft/src/arch/aarch64/context_frame.rs。
    ```rust
    pub struct Aarch64ContextFrame {
        pub gpr: [u64; 31], // 通用寄存器
        pub sp: u64,        // 栈指针
        pub elr: u64,       // 异常返回的地址。如果异常处理在EL1，则为ELR_EL1，如果异常处理在EL2，则为ELR_EL2
        pub spsr: u64,      // 存储程序状态的寄存器
    }
    ```
  - vm_system_regs
    - 用于存储guest的系统寄存器状态，VmContext实现于crates/hypercraft/src/arch/aarch64/context_frame.rs。
    ```rust
    pub struct VmContext {
    // generic timer
    pub cntvoff_el2: u64,
    cntp_cval_el0: u64,
    cntv_cval_el0: u64,
    pub cntkctl_el1: u32,
    pub cntvct_el0: u64,
    cntp_ctl_el0: u32,
    cntv_ctl_el0: u32,
    cntp_tval_el0: u32,
    cntv_tval_el0: u32,
    
    // vpidr and vmpidr
    vpidr_el2: u32,
    pub vmpidr_el2: u64,
    
    // 64bit EL1/EL0 register
    sp_el0: u64,
    sp_el1: u64,
    elr_el1: u64,
    spsr_el1: u32,
    pub sctlr_el1: u32,
    actlr_el1: u64,
    cpacr_el1: u32,
    ttbr0_el1: u64,
    ttbr1_el1: u64,
    tcr_el1: u64,
    esr_el1: u32,
    far_el1: u64,
    par_el1: u64,
    mair_el1: u64,
    amair_el1: u64,
    vbar_el1: u64,
    contextidr_el1: u32,
    tpidr_el0: u64,
    tpidr_el1: u64,
    tpidrro_el0: u64,
    
    // hypervisor context
    pub hcr_el2: u64,
    cptr_el2: u64,
    hstr_el2: u64,
    pub pmcr_el0: u64,
    pub vtcr_el2: u64,
    
    // exception
    far_el2: u64,
    hpfar_el2: u64,
    fpsimd: VmCtxFpsimd,
    pub gic_state: GicState,
    }
    ```
    - 具体可分为以下几大类别，在进入guest之前需要设置的寄存器会单独进行说明。
      - 时钟相关的寄存器
      - 多处理器相关的寄存器
        - vmpidr_el2：用于存储虚拟多处理器ID
      - guest内部需要自用的系统寄存器
        - sctlr_el1：系统控制寄存器，用于进行一些系统设置，包括内存系统
          - nTWE, bit [18]
            - 0：如果在 EL0 执行的 WFE 指令会导致执行被暂停，例如事件寄存器未设置且没有待处理的 WFE 唤醒事件，则会使用 0x1 ESR 代码将其视为对EL1 的异常。
            - 1：WFE指令正常运行。
          - nTWI, bit [16]
            - 0：如果在 EL0 执行的 WFI 指令会导致执行被暂停，例如事件寄存器未设置且没有待处理的 WFE 唤醒事件，则会使用 0x1 ESR 代码将其视为对EL1 的异常。
            - 1：WFI指令正常运行。
          - CP15BEN, bit [5]
            - 0：aarch32 CP15 barrier指令禁用，编码为UNDEFINED。
            - 1：aarch32 CP15 barrier指令启用。
          - SA0, bit [4]
            - 当该bit启用时，EL0 的加载/存储指令中堆栈指针作为基地址的使用必须对齐到 16 字节边界，否则将引发堆栈对齐故障异常。
      - hypervisor设置相关的寄存器
        - vtcr_el2：用于设置二阶段页表第二阶段翻译的相关参数
          - PS, bits [18:16]：第二阶段翻译的物理地址大小
          - TG0, bits [15:14]：VTTBR_EL2的页面粒度大小
          - SH0, bits [13:12]：用VTTBR_EL2或VSTTBR_EL2进行table walks时关联的内存的共享属性
          - ORGN0, bits [11:10]：用VTTBR_EL2或VSTTBR_EL2进行table walks时关联的内存的外部缓存属性
          - IRGN0, bits [9:8]：用VTTBR_EL2或VSTTBR_EL2进行table walks时关联的内存的内部缓存属性
          - SL0, bits [7:6]：第二阶段翻译查询的开始级别
          - T0SZ, bits [5:0]：VTTBR_EL2 寻址的内存区域的大小偏移量，区域大小为2^(64-T0SZ)字节。
        - hcr_el2：用于进行进行虚拟化的配置，比如设置哪些操作会被trap到EL2。由于目前hypervisor的实现中断控制器也以直通的方式进行虚拟化，所以未设置异常路由到EL2。
          - RW, bit [31]：
            - 0b0：Lower EL都为aarch32。
            - 0b1：在EL1的执行状态为aarch64，EL0的执行状态由PSTATE.nRW当前的值决定。
          - VM, bit [0]：
            - 0b0：禁用EL1&0 二阶段地址翻译。
            - 0b1：启用EL1&0 二阶段地址翻译。
      - 异常处理相关的寄存器
        
### 2.2.2 方法
- pub fn new(id: usize) -> Self
  - 创建一个新的VCpu。这个方法用于CPU的create_vcpu方法进行调用。
- pub fn init(&mut self, kernel_entry_point: usize, device_tree_ipa: usize)：初始化VCpu的相关寄存器
  - fn vcpu_arch_init(&mut self, kernel_entry_point: usize, device_tree_ipa: usize)
    - 设置guest trap context，将x0寄存器设置为device_tree_ipa，将elr（返回地址）设置为guest内核入口地址。
    - 设置spsr
      - SPSR_EL1::M::EL1h：设置异常级别为EL1，并且设置由EL确定用的SP是哪个EL。
      - SPSR_EL1::I::Masked：IRQ被屏蔽。
      - SPSR_EL1::F::Masked：FIQ被屏蔽。
      - SPSR_EL1::A::Masked：SError中断被屏蔽。
      - SPSR_EL1::D::Masked：目标在当前EL的Watchpoint、Breakpoint 和 Software Step 异常被屏蔽。
  - fn init_vm_context(&mut self)
    - 对sctlr_el1、vtcr_el2、hcr_el2、vmpidr_el2寄存器进行设置。（详见vm_system_regs处解释）
- pub fn vcpu_ctx_addr(&self) -> usize
  - 获取当前vcpu regs成员的地址
- pub fn run(&self, vttbr_token: usize) -> !
  - 利用hvc call进入EL2进行相关寄存器设置后，返回guest kernel entry point开始执行。详情可见[第五章](./ch5-boot-process.md)。
## 2.3 VM（crates/hypercraft/src/arch/aarch64/vm.rs）
该结构对应于guest VM运行所需的一些信息，用于存储VM底层对应的VCpus以及VM对应的页表信息。
### 2.3.1 成员变量
```rust
pub struct VM<H: HyperCraftHal, G: GuestPageTableTrait> {
    /// The vcpus belong to VM
    vcpus: VmCpus<H>,
    /// The guest page table of VM
    gpt: G,
    /// VM id
    vm_id: usize,
}
```
- vcpus: VmCpus\<H>：VM对应的VCpus。
  ```rust
    pub struct VmCpus<H: HyperCraftHal> {
        inner: [Once<VCpu<H>>; VM_CPUS_MAX],
        marker: core::marker::PhantomData<H>,
    }
  ```
  - VCpus实现于crates/hypercraft/src/vcpus.rs，定义了包含多个VCpu的数组。目前尚未实现多VCpu的支持。
- gpt: G：VM的Guest Page Table，即二阶段翻译中第二阶段（ipa → hpa）对应的页表。具体详情会在[第四章](./ch4-memory-virtualization.md)中介绍。
- vm_id: usize：VM对应的id。
### 2.3.2 方法
- pub fn new() -> Self
  - 创建一个新的VM结构。
- pub fn init_vm_vcpu(&mut self, vcpu_id:usize, kernel_entry_point: usize, device_tree_ipa: usize)
  - 找到对应vcpu_id的vcpu，调用vcpu的init方法，将guest kernel入口点以及dtb的ipa作为参数传入，初始化vcpu的相关寄存器设置。
- pub fn run(&mut self, vcpu_id: usize)
  - 运行该VM。获取对应vcpu_id的vcpu，并且找到guest page table的基址，作为参数传给vcpu的run方法，启动该虚拟机。具体在[第五章](./ch5-boot-process.md)中进行介绍。
## 2.4 练习
1. 在合适的地方修改代码，打印出在vcpu初始化前后regs中guest_trap_context_regs与vm_system_regs各个寄存器的值。比较它们哪些值发生了变化。并根据本章节介绍的内容，对照[Arm A-profile Architecture Registers](https://developer.arm.com/documentation/ddi0601/latest/)，说明值变化的寄存器的作用。