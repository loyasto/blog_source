---
title: linux的网络设备驱动分析
date: 2016-12-03 14:06:22
tags:
	- linux
	- 网络
---
网络设备是linux里与字符设备、块设备并列的3种设备类型之一。
与字符设备和块设备不同，网络设备在/dev目录下没有节点。网络设备不太符合“一切都是文件”的思想。


从全局来看，Linux对网络驱动的框架体系设计如下，分为4层。
```
-------------------
协议接口层。这个就是跟tcpip对接的那一层，提供一个发送一个接收函数给tcpip调用。
-------------------
设备接口层。核心就是net_device这个结构体。
-------------------
设备驱动实现层。这个就是写驱动的时候，主要做的事情，在这里实现一堆函数，通过net_device注册到内核里。
-------------------
媒介层。
-------------------
```
# 1. 协议接口层
核心数据结构是`sk_buff`。是socket buffer的意思。
这个结构体可以叫做Linux网络子系统的中枢神经，网络数据在各层之间传递全靠它了。
针对这个结构体，Linux提供一些接口来进行操作，如下：
分配：`alloc_skb`和`dev_alloc_skb`。对应的释放函数`kfree_skb`和`dev_kfree_skb`，以及`dev_kfree_skb_irq`和`dev_kfree_skb_any`。
驱动里建议使用以dev开头的接口。
另外还有修改的接口。
`skb_put`：这个是往tail上增加数据。
`skb_push`：这个是往head上增加数据。
`skb_reserve`：这个是把数据整个往tail的方向挪动一段，在前面腾出一些空间出来。

# 2. 设备接口层
这一层的核心结构体是net_device。
net_device实现了对所有网络设备的抽象。达到对上提供统一接口的目的。
结构体内容较多，定义的代码有300行左右。我们总结重要的如下：
1. 全局信息类。
  有个name字符数组。
2. 硬件信息类。
  共享内存的起始和结束地址、中断号。
3. 接口信息类。
  就是包头长度，MTU值，设备地址这些。
4. 设备操作函数。
5. 其他。


# 3. 以dm9000为例来分析

dm9000的硬件特点：

1、有100个引脚。

2、数据位宽是16位的。

3、只有一根地址线。

4、支持10M/100M模式。

dm9000的寄存器有：从0x00到0xff。

0x00：NCR。控制寄存器。

不一一去看了。



我们写驱动，驱动文件里写的都是platform_driver。而这个driver里面的函数的参数就是platform_device。

就是这样联系起来的。



入口函数，非常简单如下：

```
static int __init
dm9000_init(void)
{
	printk(KERN_INFO "%s Ethernet Driver, V%s\n", CARDNAME, DRV_VERSION);

	return platform_driver_register(&dm9000_driver);
}

static void __exit
dm9000_cleanup(void)
{
	platform_driver_unregister(&dm9000_driver);
}
```
以太网设备属于平台设备，所以其初始化和去初始化函数，就是平台设备的注册和反注册。
入口函数涉及的结构体是`platform_driver`，定义一个变量如下：
```
static struct platform_driver dm9000_driver = {
	.driver	= {
		.name    = "dm9000",
		.owner	 = THIS_MODULE,
		.pm	 = &dm9000_drv_pm_ops,
	},
	.probe   = dm9000_probe,
	.remove  = __devexit_p(dm9000_drv_remove),
};
```
所以需要实现的总共是4个函数：
probe、remove、suspend、resume。
probe相当于初始化函数，我们重点分析一下。
probe函数原型是：`int (*probe)(struct platform_device *);`。`platform_driver`和`platform_device`是如何关联起来的呢？

过程是这样的：`platform_driver`里面有一个`device_driver`，`device_driver`有2个需要注意的成员变量name和owner。写驱动的时候，一般是把owner写出`THIS_MODULE`。
然后在`platform_device`的name与`device_driver`的name值一样的时候，就被关联起来了。



probe函数的唯一参数是platform_device。怎样由这个参数得到其他的结构体呢？

1、dm9000_platform_data。这个是在include/linux/dm9000.h这个头文件里。（这个头文件都放到这个层级的目录里了，算是Linux系统的一部分了）

平台设备的相关内容是在arch/arm/mach-s3c2410/mach-bast.c里。

看文件里的信息，这个文件属于某个板子，这块板子是德国的simtec公司的，这家公司是做运动模拟系统的。



## 3.1 dm9000_probe函数分析

输入：platform_device *pdev

主要要操作的结构体：

net_device。



1、通过pdev->dev.platform_data得到dm9000_plat_data。

2、用alloc_etherdev函数来得到struct net_device *ndev。

​	这个函数的参数，传递的就是私有数据的大小。

3、把ndev的dev的parent设置为pdev->dev。这样就把net_device和platform_device联系起来了。

4、把struct board_info * bi = netdev_priv(ndev)。来取到私有数据的地址。

​	私有数据里有一个dev指针，一个ndev指针。

5、 初始化delayed_work，这个是用来处理phy状态的，应该就是网线的插拔处理用。

6、ioremap一些资源。

7、dm9000_set_io。这个是设置是32位还是16位的数据宽度。

8、dm9000_reset。写一些寄存器。

9、读取dm9000的id，如果不对，就报错。然后根据id，设置一些配置。

10、调用ether_setup。就是设置net_device参数，例如包长度，类型等。

11、ndev的一些重要成员赋值。netdev_ops。ethtool_ops

12、从eeprom读取mac地址。

13、platform_set_drvdata。把pdev和ndev联系起来。

14、register_netdev。

## 3.2 net_device_ops分析

