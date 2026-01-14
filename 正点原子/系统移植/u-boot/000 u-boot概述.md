# u-boot概述

## 获取u-boot
https://ftp.denx.de/pub/u-boot/

## 什么是u-boot
- 一个裸机程序
- 一个bootloader

## u-boot的作用
- 主要是初始化DDR内存
    - Linux在DDR中运行
    - zImage/dtb存放在EMMC/SD卡/NAND/SPI FLASH等外置存储设备
- 用于启动Linux/其他OS
    - 提供外置存储设备的驱动程序
    - 拷贝系统镜像到DDR内存中
