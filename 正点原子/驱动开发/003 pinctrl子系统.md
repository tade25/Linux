# pinctrl子系统
- 设置引脚的复用和电气属性

```dts
// imx6ull.dtsi

iomuxc: iomuxc@020e0000 {
    compatible = "fsl,imx6ul-iomuxc";
    reg = <0x020e0000 0x4000>;
};
```

```c
#include "imx6ul-pinfunc.h"

// <mux_reg conf_reg input_reg mux_mode input_val>

#define MX6UL_PAD_UART1_RTS_B__GPIO1_IO19       0x0090 0x031C 0x0000 0x5 0x0
```

```dts
&iomuxc {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_hog_1>;
	imx6ul-evk {
		pinctrl_hog_1: hoggrp-1 {
			fsl,pins = <
				MX6UL_PAD_UART1_RTS_B__GPIO1_IO19       0x17059 /* SD1 CD */
				MX6UL_PAD_GPIO1_IO05__USDHC1_VSELECT    0x17059 /* SD1 VSELECT */
				MX6UL_PAD_GPIO1_IO09__GPIO1_IO09        0x17059 /* SD1 RESET */
			>;
		};
        ...
    }
};
```

- 复用的地址 = 0x020e0000 + 0x0090
- *(0x020e0000 + 0x0090) = 0x5
- 电气属性的地址 = 0x020e0000 + 0x031C
- *(0x020e0000 + 0x031C) = 0x17059

- pinctrl驱动
    - pinctrl的驱动不同的芯片是不一样的
    - drivers/pinctrl/freescale/pinctrl-imx6ul.c，imx6ul/imx6ull的pinctrl驱动文件
