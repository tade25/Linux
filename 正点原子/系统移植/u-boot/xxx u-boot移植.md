# u-boot移植

## 1. 编译imx的开发板与测试
```bash
# 安装依赖
sudo apt install build-essential

# 配置环境变量
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-

# 清理工程
make distclean

# 配置
make xxx_defconfig #比如 mx6ull_14x14_evk_emmc_defconfig

# 编译u-boot
make -j4

# 编译过程会自动向u-boot.bin添加头部信息(通过./tools/mkimage)生成u-boot.imx

# 查看sd卡
sudo fdisk --list

# 烧写u-boot
./imxdownload u-boot.bin /dev/sdb

```

## 2. 添加自己的开发板
### 2.1 添加板子的配置文件
```bash
    cp configs/mx6ull_14x14_evk_emmc_defconfig configs/mx6ull_tade_emmc_defconfig
```

### 2.2 添加板子的头文件
不同板子的配置信息，在不同的头文件里面配置
```bash
    cp include/configs/mx6ullevk.h include/configs/mx6ull_tade_emmc.h
```

### 2.3 添加板子的板级文件夹
- 修改文件名称
- 修改Makefile
- 修改imximage.config
- 修改Kconfig
- 修改MAINTAINERS

### 2.4 修改u-boot的配置界面
/home/tan/My_Linux/uboot-imx-rel_imx_4.1.15_2.1.0_ga/arch/arm/cpu/armv7/mx6

## 3. 修改LCD驱动
- LCD的GPIO是否正确(lcd_pads)
- 背光引脚
- LCD参数(displays)
- 头文件里面的TFTE3AB
- panel环境变量

## 4. 修改网络驱动
- 修改PHY ADDR
- 删除原有的74LV595代码
- 添加板子的网络复位IO
- 设置ipaddr/ethaddr(00:04:9f:04:d2:35)环境变量

## 5. 启动Linux内核
- 从emmc启动
    - 是否有系统/Linux镜像/zImage、dtb
    - mmc dev 1/切换到emmc
    - mmcinfo/查看emmc信息
    - fatls mmc 1:1/查看emmc分区1的内容
    - fatload mmc 1:1 80800000 zImage/加载zImage到0x80800000
    - fatload mmc 1:1 83000000 imx6ull-14x14-emmc-7-1024x600-c.dtb/加载dtb到0x83000000
