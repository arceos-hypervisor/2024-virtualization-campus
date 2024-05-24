## 运行
假设我们以下操作都是在$(WORKSPACE)目录下进行操作。
### 1.1 编译linux镜像
1.4.2节运行的系统为linux。为了编译这个OS内核，执行以下指令：
```shell
# set up cross-compile tools
# download
wget https://musl.cc/aarch64-linux-musl-cross.tgz
# install
tar zxf aarch64-linux-musl-cross.tgz
# exec below command in bash
export PATH=`pwd`/aarch64-linux-musl-cross/bin:$PATH
# OR add below info in ~/.bashrc
# echo PATH=`pwd`/aarch64-linux-musl-cross/bin:$PATH >> ~/.bashrc

# download and unzip linux kernel
wget https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/snapshot/linux-6.2.15.tar.gz
tar -xvf linux-6.2.15.tar.gz
cd linux-6.2.15

# build linux
mkdir build
make O=build ARCH=arm64 CROSS_COMPILE=aarch64-linux-musl- defconfig
make O=build ARCH=arm64 CROSS_COMPILE=aarch64-linux-musl- #-j4 
```
### 1.2 构建文件系统
```shell
# use busybox to build rootfs
# download
wget https://busybox.net/downloads/busybox-1.36.1.tar.bz2
tar xvf busybox-1.36.1.tar.bz2

# compile busybox
cd busybox-1.36.1
mkdir build
make O=build ARCH=arm64 defconfig
make O=build ARCH=arm64 menuconfig
## select and save the following settings
## Settings -> [*] Don't use /usr
## Settings -> [*] Build static binary (no shared libs)
## Settings -> (aarch64-linux-musl-) Cross compiler prefix
make O=build #-j4
make O=build install

# build rootfs
cd build/_install && mkdir -pv {etc,proc,sys,dev,usr/{bin,sbin}}
cd ..
## create a image
dd if=/dev/zero of=rootfs.img bs=1M count=512 
## format filesystem
mkfs.ext4 rootfs.img
## mount filesystem
mkdir tmp
sudo mount rootfs.img tmp
## copy and create the content of the filesystem to mount point
sudo cp -r _install/* tmp/
cd tmp/dev
sudo mknod console c 5 1
sudo mknod null c 1 3
sudo mknod tty1 c 4 1 
sudo mknod tty2 c 4 1 
sudo mknod tty3 c 4 1 
sudo mknod tty4 c 4 1 
## create /etc/fstab
cd ../etc
sudo vim fstab
## copy following content to fstab
proc     /proc                   proc     defaults        0 0
sysfs    /sys                    sysfs    defaults        0 0
## create /etc/init.d/rcS
sudo mkdir init.d && cd init.d
sudo vim rcS
## copy following content to rcS
#!/bin/sh
echo -e "Welcome to arceos Linux"
mount -a
echo -e "Remounting the root filesystem"
## create /etc/inittab
cd ..
sudo vim inittab
## copy following content to inittab
# /etc/inittab
::sysinit:/etc/init.d/rcS
console::respawn:-/bin/sh
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r
## umount filesystem
cd ../../
sudo umount tmp
```
### 1.3 编译hypervisor
```shell
# clone arceos
git clone https://github.com/arceos-hypervisor/arceos.git -b multicore-aarch64 --recurse-submodules
cd arceos

# set up rust tools
cargo install cargo-binutils

# build and run arceos aarch64 hypervisor
make A=apps/hv ARCH=aarch64 HV=y LOG=info SMP=2 build
```
### 1.4 运行hypervisor
在qemu中运行arceOS，并在arceOS上起虚拟机。在0号核（主核）上启动Linux虚拟机，在1号核（副核）上启动Nimbos虚拟机。
```shell
# move nimbos image to arceos hv 
cp $(WORKSPACE)/nimbos/kernel/target/aarch64/release/nimbos.bin apps/hv/guest/nimbos/nimbos-aarch64.bin

# move linux image and rootfs to arceos hv 
cp $(WORKSPACE)/linux-6.2.15/build/arch/arm64/boot/Image $(WORKSPACE)/arceos/apps/hv/guest/linux/linux-aarch64.bin
cp $(WORKSPACE)/busybox-1.36.1/build/rootfs.img $(WORKSPACE)/arceos/apps/hv/guest/linux/rootfs-aarch64.img

# run linux(virtio)
qemu-system-aarch64 -m 3G -smp 2 -cpu cortex-a72 -machine virt -kernel apps/hv/hv_qemu-virt-aarch64.bin -device loader,file=apps/hv/guest/linux/linux-multi-core.dtb,addr=0x70000000,force-raw=on -device loader,file=apps/hv/guest/linux/linux-aarch64.bin,addr=0x70200000,force-raw=on -machine virtualization=on,gic-version=2 -drive if=none,file=apps/hv/guest/linux/rootfs-aarch64.img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0  -nographic
```

## 注意
- 目前还未完整编写好启动脚本，所以直接用qemu命令启动，修改完成后重新编译dts到dtb，多核对应的dts文件为aarch64-test.dts。
