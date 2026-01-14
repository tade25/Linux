# u-boot命令

## 基础命令
- **help / ?**
    - 帮助信息
    - 查看某一个命令的帮助信息: ? boot

- **bdinfo**(板子信息)

- **printenv**(查看当前板子的环境变量)

- **setenv**(设置环境变量)
    - setenv bootdelay 5
    - setenv bootcmd 'console=ttymxc0,115200 root=/dev/mmcblklp2 rootwait rw'
- **saveenv**(保存环境变量)

## 内存命令
- **md**(显示内存值)
    - md.[b,w,l] address [#of objects]
    - address，内存的起始地址
    - [#of objects], 需要查看的长度

- **nm**(修改指定地址的内存值)
    - nm.[b,w,l] address
    - address，要修改的内存地址

- **mm**(连续的修改指定地址的内存值)
    - mm.[b,w,l] address
    - address，要修改的内存地址

- **mw**(使用一个指定的数据填充一段内存)
    - mw.[b,w,l] address value [count]
    - address，要填充的起始地址
    - value，要填充的数据
    - count，要填充的长度

- **cp**(数据拷贝命令，将DRAM中的数据从一段内存拷贝到另一段内存)
    - cp.[b,w,l] source target count
    - source，源地址
    - target，目标地址
    - count，拷贝的长度

- **cmp**(比较命令，比较两段内存的数据是否相等)
    - cmp.[b,w,l] addr1 addr2 count
    - addr1/addr2，第一段/第二段内存首地址
    - count，比较的长度

## 网络命令
```bash
# 设置ipaddr
setenv ipaddr 192.168.191.30

# 设置ethaddr
setenv ethaddr 00:04:9f:04:d2:35

# 设置gatewayip
setenv gatewayip 192.168.191.1

# 设置netmask
setenv netmask 255.255.255.0

# 设置服务器ip地址/ubuntu的ip地址
setenv serverip 192.168.191.135

# 保存环境变量
saveenv
```

- **ping**(测试网络的连通性)
    - ping ping_addr

- **dhcp**(获取ip地址)

- **nfs**(网络调试，方便调试Linux镜像和设备树)
    - nfs [local_address] [host_ip_addr:boot_file_name]
    - nfs 80800000 192.168.191.135:/home/tan/nfs_share/zImage

- **tftp**
    - ubuntu安装tftp-hpa/tftpd-hpa
    - ubuntu创建一个和nfs一样的目录
    - ubuntu给予超级权限

## MMC命令
- **mmc info**(当前选中的设备的信息)

- **mmc rescan**(扫描mmc设备)

- **mmc list**(列出可用的mmc设备)

- **mmc dev [dev] [part]**(列出/设置当前的mmc设备)
    - mmc dev 0

- **mmc part**(列出当前mmc设备的分区)

- **mmc read**(读取mmc设备的数据)
    - mmc read addr blk#  cnt
    - mmc read 80800000 600 10

- **mmc write**(将数据写入mmc设备)
    - mmc write addr blk#  cnt
    - mmc write 80800000 600 10
