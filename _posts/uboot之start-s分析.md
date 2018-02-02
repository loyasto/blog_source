---
title: uboot之start.s分析
date: 2017-08-15 23:41:16
tags:

	- uboot

---

本文是对https://www.crifan.com/files/doc/docbook/uboot_starts_analysis/release/htmls/index.html的学习总结。感谢这位热心网友的无私奉献。

本文目标：

1、可以对start.s的每一行代码都有认识。

2、可以对ARM核的S3C24X0的CPU启动过程有深入理解。

分析的代码基础是天嵌的2440板子的uboot源代码。

# 1. 全局了解

start.s主要做的就是系统各个方面的初始化。主要是下面几件事情：

1、设置cpu模式。

2、关闭看门狗。

3、关闭中断。

4、设置堆栈sp指针。

5、清楚bss段。

6、异常中断处理。

# 2. start.S逐句分析

1、`.global:_start`

可以参考《gas ARM Ref》文档。地址在：

[http://re-eject.gbadev.org/files/GasARMRef.pdf](http://re-eject.gbadev.org/files/GasARMRef.pdf)

`.global`相当于c语言里的extern声明，说明这个`_start`符号是全局的。

这个_start在u-boot.lds里有用到：

```
ENTRY(_start)
```

继续看。`_start:`后面带冒号就表示是一个label。这里就是最后链接出来的东西的开始的位置。

如果是从nor flash里启动，那么这个地址就是真的是物理地址0 。

如果relocation之后，就是我们配置的内存里的某个地址。就是uboot里的TEXT_BASE这个变量。

例如是0x33d0 0000这样的值。

```
_start:
	b reset
```

继续。

```
.balignl 16, 0xdeadbeef
```

就是说要16字节对齐，如果没有对齐，就用0xdeafbeef这个数据来占用几个字节的空间。



