# platform总线

## 一、platform总线存在的原因

### 1.1 Linux 的驱动设计哲学
Linux内核它只关心：(1)设备是什么 (2)谁来驱动它 (3)怎么撮合它们，不关心是谁家的芯片，于是衍生出了**bus-device-driver模型**

### 1.2 platform总线解决了什么问题？
- Soc不全是PCI / USB / I2C这种“插上就枚举”的总线 ，还有UART / GPIO / I2C Controller / SPI Controller / DMA / Timer / RTC这类没有总线协议枚举的
- Linux引入一个概念platform：一种“虚拟总线”，专门承载SoC内部设备

## 二、总线-设备-驱动模型
- 驱动逻辑层, device_driver
- 总线核心层，bus_type
    - 总线的工作是完成总线下的设备和驱动之间的匹配
    - /sys/bus
    - 向Linux内核注册总线：bus_register
- 设备描述层，device
```mermaid
graph BT
A[设备描述层 --- 来自DT、代码等] --> B[总线核心层 --- match / probe调度]
B[总线核心层 --- match / probe调度] --> C[驱动逻辑层 --- probe / remove / pm]
```

## 三、bus_type
```c
#include <linux/device.h>

struct bus_type {
    const char *name;
    int (*match)(struct device *dev, struct device_driver *drv);
    int (*probe)(struct device *dev);
    int (*remove)(struct device *dev);

    const struct dev_pm_ops *pm;
    ... // 不重要的内容
};
```

## 四、device_driver
```c
#include <linux/device.h>

struct device_driver {
    const char *name;
    struct bus_type *bus;

    const struct of_device_id *of_match_table;

    int (*probe)(struct device *dev);
    int (*remove)(struct device *dev);
    ... // 不重要的内容
};
```
driver_register向总线注册驱动，会检查当前总线下的所有设备，有没有与此驱动匹配的设备，如果有就执行驱动里的probe函数

## 五、device
```c
#include <linux/device.h>

struct device {
    struct bus_type *bus;
    struct device_driver *driver;

    struct device_node *of_node;
    void *driver_data;
};
```

## 六、platform总线
- platform总线是bus_type的一个实例
```c
struct bus_type platform_bus_type = {
	.name	= "platform",
	.match	= platform_match,
	.uevent	= platform_uevent,
	.pm	= &platform_dev_pm_ops,
};
```

## 七、platform_device
- platform_device = device + SoC资源信息
```c
#include <linux/platform_device.h>

struct platform_device {
    const char *name;
    int id;

    struct resource *resource;
    struct device dev;
    ... // 不重要的内容
};
```

- platform_device的来源
    - 设备树，详情见第9章DT到platform_device的真相
    - 老派写法，如下所示
```c
#include <linux/platform_device.h>

static struct resource uart_res[] = {
    {
        .start = 0x10000000,
        .end   = 0x10000fff,
        .flags = IORESOURCE_MEM,
    },
    {
        .start = 32,
        .end   = 32,
        .flags = IORESOURCE_IRQ,
    },
};

static struct platform_device uart_pdev = {
    .name = "my_uart",
    .id   = -1,
    .num_resources = ARRAY_SIZE(uart_res),
    .resource = uart_res,
};

platform_device_register(&uart_pdev);
```

## 八、platform_driver
- platform_driver，并没有多加功能，只是把device_driver包在结构体
```c
#include <linux/platform_device.h>

struct platform_device_id {
    char name[PLATFORM_NAME_SIZE];
    kernel_ulong_t driver_data;
};

struct platform_driver {
    int (*probe)(struct platform_device *);
    int (*remove)(struct platform_device *);

    struct device_driver driver;
    const struct platform_device_id *id_table;
    ... // 不重要的内容
};
```

- platform_driver的注册函数
```c
int platform_driver_register(struct platform_driver *drv);
```

## 九、DT -> device_node -> platform_device的真相
- 1、内核启动早期
```text
start_kernel
└─ setup_arch
    └─ unflatten_device_tree
        └─ __unflatten_device_tree
```

```c
//==============================
// 1) 全局关键变量
//==============================
struct device_node *of_root;        // 解析后的设备树根节点 ("/")
void *initial_boot_params;          // Bootloader 传进来的原始 DTB 起始地址 (FDT blob)

// initial_boot_params 指向的是一块 "flattened device tree" (DTB) 的内存
// 里面是 FDT 格式：header + structure block + strings block

//==============================
// 2) 把 DTB -> device_node 树
//==============================
void __init unflatten_device_tree(void)
{
    __unflatten_device_tree(initial_boot_params,
                            NULL,
                            &of_root,
                            early_init_dt_alloc_memory_arch,
                            false);
}

// blob      : 原始 dtb 的起始地址 (FDT blob)
// dad       : 父节点(这里传 NULL，表示从根开始构建整棵树)
// mynodes   : 输出参数，返回解析出来的根 device_node (of_root)
// dt_alloc  : 内存分配器 (early 阶段用 memblock 分配)
// detached  : 是否标记为 detached tree
static void *__unflatten_device_tree(const void *blob,
                                     struct device_node *dad,
                                     struct device_node **mynodes,
                                     void *(*dt_alloc)(u64 size, u64 align),
                                     bool detached)
{
    int size;
    void *mem;

    // 0) 检查 blob 是否有效
    // fdt_check_header(blob) 验证 dtb header/magic 等

    //==================================================
    // 1) 第一遍：dry-run 只遍历，计算需要多少内存
    //==================================================
    size = unflatten_dt_nodes(blob, NULL, dad, NULL);
    // mem == NULL -> dryrun = true
    // 只算 device_node + property + 字符串等需要的总大小

    size = ALIGN(size, 4);

    //==================================================
    // 2) 分配一整块连续内存 (early 阶段)
    //==================================================
    mem = dt_alloc(size + 4, __alignof__(struct device_node));

    memset(mem, 0, size);

    //==================================================
    // 3) 第二遍：real-run 真正创建节点并填充数据
    //==================================================
    unflatten_dt_nodes(blob, mem, dad, mynodes);
    // mem != NULL -> dryrun = false
    // 在 mem 这块连续内存里依次摆放并填充：
    //   struct device_node
    //   struct property
    // 并建立 parent/child/sibling/property 链表关系

    return mem;
}

//==================================================
// unflatten_dt_nodes() 的核心意义
//==================================================
//
// 第一次调用：计算大小（dryrun）
//   unflatten_dt_nodes(blob, NULL, ...)
//     -> 遍历 dtb 所有 node
//     -> 统计需要多少内存
//
// 第二次调用：构建树（real-run）
//   unflatten_dt_nodes(blob, mem, ...)
//     -> 真的生成 device_node 树
//     -> 最终得到 of_root 指向根节点 "/"
//
//==================================================
```
👉 最终效果：把.dtb(flattened FDT)解析成一棵device_node树(of_root)

- 2、platform总线初始化
```text
platform_bus_init
 └─ bus_register(&platform_bus_type)
```

- 3、DT生成platform_device
```text
start_kernel
└─ rest_init
    └─ kernel_init
        └─ kernel_init_freeable
            └─ do_basic_setup
                └─ do_initcalls
                    └─ ...
                        └─ of_platform_populate

of_platform_populate()
 └─ 对每个 compatible 的 DT 节点
     └─ platform_device_alloc()
     └─ pdev->dev.of_node = node
     └─ platform_device_add()
         └─ device_register()
```
👉 此时已经有了：
- platform_device
- dev.of_node
- dev.bus = platform_bus_type

- 4、platform_driver注册
```text
platform_driver_register()
 └─ driver_register()
     └─ bus_for_each_dev()
         └─ platform_match()
             └─ of_driver_match_device()
                 └─ 匹配 compatible
                     └─ 调用 probe
```

## 九、platform_match的优先级
匹配顺序 非常重要：
1. driver_override（强制绑定）
2. of_match_table（设备树，最常用）
3. acpi_match_table
4. id_table
5. pdev->name == drv->name
