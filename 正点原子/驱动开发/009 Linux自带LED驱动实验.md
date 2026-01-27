# Linux自带的LED驱动

## 1. 将驱动编译进内核
- make menuconfig
```text
 └─ Device Drivers
     └─ LED Support
         └─ LED Support for GPIO connected LEDs
```
- 配置之后.config就会存在CONFIG_LEDS_GPIO，即
```
CONFIG_LEDS_GPIO=y
```

- /drivers/leds/Makefile
```text
obj-$(CONFIG_LEDS_GPIO) += leds-gpio.o，那么
obj-y += leds-gpio.o
```

## 2. 根据绑定文档添加设备树节点
- Documentation/devicetree/bindings/leds

