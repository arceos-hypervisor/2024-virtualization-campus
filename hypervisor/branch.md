# ArceOS-Hypervisor分支说明

## hypervisor 
2023年秋冬训练营使用的分支。

x86_64架构：能够支持单核单vm的nimbos系统启动。具体运行方式参见[x86_64 nimbos](https://github.com/arceos-hypervisor/hypercraft?tab=readme-ov-file#x86_64-nimbos)

aarch64架构：能够支持单核单vm的nimbos与linux系统的启动。具体运行方式参见[runcode](./aarch64/ch1-el-architecture-runcode.md)

riscv架构：能够支持单核单vm的linux系统启动。具体运行方式参见[Riscv Linux](https://github.com/arceos-hypervisor/hypercraft?tab=readme-ov-file#riscv-linux)

## process

x86_64架构：系统调用转发分支。能够支持一个linux和一个process运行，支持转发open/read/write/close四个syscall。具体运行方式：**TODO**

## boot_linux 

x86_64架构：能够支持多核多VM的Linux系统启动。基于type1.5的方式启动ubuntu，在完成ubuntu系统降级后，能够启动第二个Linux系统。具体运行方式参见[How-to-boot-VM](https://github.com/arceos-hypervisor/arceos/blob/boot_linux/How-to-boot-VM.md)

## hypervisor-aarch64-dev 

aarch64架构：能够支持两个单核Linux系统的VM启动（initramfs）。具体运行方式参见[multi-vm](./aarch64/multi-vm.md)

## multicore-aarch64

aarch64架构：能够支持单个两个核的Linux系统的VM启动。具体运行方式参见[multi-core](./aarch64/multi-core.md)
