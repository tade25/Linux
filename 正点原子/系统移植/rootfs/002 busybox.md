# busybox配置

下载地址：https://busybox.net/

## 使用busybox制作根文件系统
```bash
# 修改Makefile，添加交叉编译工具链
CROSS_COMPILE ?= /usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-

# 添加中文支持
libbb/printable_string.c
函数printable_string
    屏蔽if(c >= 0x7f) break;
    if(c < ' ' || c >= 0x7f) 改成 if(c < ' ')

libbb/unicode.c
函数unicode_conv_to_printable2
    *d++ = (c >= ' ' && c < 0x7f) ? c : '?'
    改成
    *d++ = (c >= ' ') ? c : '?'

    if(c < ' ' || c >= 0x7f)
    改成
    if(c < ' ')

# 使用默认配置
make defconfig

# 打开图形化配置界面
make menuconfig

settings
    build static binary # 不选中，静态/动态库
    vi-style line editing commands # 选中
    support unicode # 选中
        check SLC_ALL,SLC_CTYPE and SLANG envirnment variables # 选中 

Linux Module Utilities
    simplified modutils # 取消勾选

Linux System Utilities
    mdev(16kb) # 确定这个选项下的都被选中

# 编译busybox
make install CONFIG_PREFIX=~/My_Linux/nfs/rootfs

# 拷贝库文件
mkdir lib
cd /usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/libc/lib
cp -d *so* *.a ~/My_Linux/nfs/rootfs/lib
rm -rf ld-linux-armhf.so.3 # ld-linux-armhf.so.3不能是链接文件，搞不懂
cp ld-linux-armhf.so.3 ~/My_Linux/nfs/rootfs/lib

cd /usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/lib
cp -d *so* *.a ~/My_Linux/nfs/rootfs/lib

mkdir usr/lib
cd /usr/local/arm/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/libc/usr/lib
cp -d *so* *.a ~/My_Linux/nfs/rootfs/usr/lib

# 使用du查看文件大小
cd rootfs
du ./lib/ ./usr/lib -sh

# 创建其他文件夹
mkdir etc dev proc mnt sys tmp root

# 创建/etc/init.d/rcS
cd /etc
mkdir init.d
cd init.d
vi rcS

输入
#!/bin/sh

PATH=/sbin:/bin:/usr/sbin:usr/bin:$PATH
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/lib:/usr/lib
export PATH LD_LIBRARY_PATH

mount -a
mkdir /dev/pts
mount -t devpts devpts /dev/pts

echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s

# 创建/etc/fstab
cd /etc
vi fstab

输入
#<file system>  <mount point>   <type>  <options>   <dump>  <pass>
proc            /proc           proc    defaults    0       0
tmpfs           /tmp            tmpfs   defaults    0       0
sysfs           /sys            sysfs   defaults    0       0

# 创建/etc/inittab
<id>:<run_levels>:<action>:<process>
cd /etc
vi inittab

输入
#/etc/inittab
::sysinit:/etc/init.d/rcS
console::askfirst:-/bin/sh
::restart:/sbin/init
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r
::shutdown:/sbin/swapoff -a

# 创建/etc/resolv.conf
vi resolv.conf
输入
nameserver 114.114.114.114
nameserver 192.168.191.1
```

# 开机自启动
```bash
vim /etc/init.d/rcS
cd xxx
./xxx &
```

## 测试-挂载网络文件系统
- 要求kernel网络驱动正常
- 设置u-boot的bootargs参数

```bash
# 修改u-boot的bootargs参数
setenv bootargs 'console=ttymxc0,115200 root=/dev/nfs nfsroot=<server_ip>:<root_dir> ip=<client_ip>:<server_ip>:<gw-ip>:<netmask>:<host_name>:<device>:<autoconf>:<dns0_ip>:<dns1_ip>'
saveenv

# 举例
setenv bootargs 'console=ttymxc0,115200 root=/dev/nfs rw nfsroot=192.168.191.140:/home/tan/My_Linux/nfs/rootfs/,nfsvers=3,proto=tcp ip=192.168.191.130:192.168.191.140:192.168.191.1:255.255.255.0::eth0:off'
saveenv
```
