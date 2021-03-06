---
title: Linux内核之内存管理buddy和slab
date: 2018-03-18 13:57:09
tags:
	- Linux
typora-root-url: ..\
---



讨论buddy（伙伴系统）和slab之前，我们想要理解内存碎片的概念。

内存碎片分为两种：

1、内部碎片。

```
内部碎片就是，你分配43个字节，而系统一般会对齐，给你44或者48字节。然后其实就有1个字节或者5个字节，你没有用上，而其他人也没法用的。除非你释放了。这种还好。
```

2、外部碎片。

```
就是一般说的内存碎片了。虽然还有，但是尺寸太小没法用。有些进程是需要申请连续的内存，所以就会分配失败。
```

# buddy系统

buddy就是为了解决外部碎片的问题。

buddy算法1963发明的。容易实现。

它把内存按照2的幂大小排成链表队列。放在free_area数组里。

```
strurct zone {
  ...
  struct free_area	free_area[MAX_ORDER];
}
```

![Linux内核之内存管理buddy和slab](/images/Linux内核之内存管理buddy和slab-图1.png)



假设我们的系统的ram只有16个page。

那么我们的buddy系统只需要4个order就可以包含进来了。

```
1、16个4K的页面，是一个整体。
2、程序A请求一块3K的内存。一个order 0的就可以满足。
	2.1、现在没有order 0的块。所以整块内存被切分两半。得到2个order3的。
	2.2、还是没有order 0的，继续切分前面的一半为4个2个page的块。
	2.3、还是没有order 0的，继续切分。现在最前面已经分出来2个order 0的了。
		分配给A。
3、现在程序B要5K的内存，有order1的在，直接分配给它。
4、程序C要3K的内存。有order 0的在，分配给它。
5、现在程序D要6K的内存，现在没有存在的可用的order1 的。
	6.1 
```

在释放内存的过程中，会看挨着的内存块是否是空闲的且跟自己是兄弟关系（order相同，存在page的private里了）。是的话，就合并。

就是这个自动的合并，解决了内存碎片。





单一的伙伴算法需要通过别的算法来提升碎片合并和提高回收速度。

位图算法就是其中一个。

# slab

采用伙伴算法分配内存的时候，每次至少要一个page，就是4K。

但是在内核里，进程会kmalloc一些比较小的结构体，几个字节的。

怎么满足这种需求，而且又能保证内核里不至于内存碎片化呢？

于是就有了slab。

linux里用的slab是从sunos里借鉴的。

sunos里的slab是Jeff Bonwick写的。

Jeff经过分析发现，在内核里，总是在为那么几种类型分配了大量的内存。

（我觉得也是二八原则，80%的内存是分配给20%种类的对象了）。

而Jeff统计发现，内核里普通对象的初始化要用的时间比分配释放内存用的时间还要多，这怎么行。

有问题，就有优化的空间。

Jeff给出的解决方案是：这些被初始化过的内存对象在释放不要被释放回内存池。标记为废弃。后面需要时，直接用。就不需要重新初始化了。

## slab分类

现在linux里的slab实际上有三套实现：

1、slab。古老。过时。不建议。

2、slub。优化版本。建议。

3、slob。嵌入式版本。

我当前的mylinuxlab里的配置：

```
# CONFIG_SLAB is not set
CONFIG_SLUB=y
```

但是网上很多文章都是分析slab的。所以我也还是先从slab入手吧。





#地址映射

开了mmu，跟不开mmu，在内存访问速度上，还是有差异的。

mmu保证了安全性和稳定性，牺牲了一定的性能。

通过虚拟地址访问到物理内存地址上，中间读了几次内存，而且还有几次加法运算。

虽然有cache辅助，还是比不上直接访问内存的方式。

所示linux的实时性不好，要实时性，就要用平板内存。

# 虚拟地址管理

每一个进程对应一个task结构体。

每个task结构体有一个mm_struct指针。

这个指针就是进程的内存管理。

（每个线程也都有一个mm_struct，但是它们指向同一个地址）。

当进程被调度的时候，页表被切换。

（对于cpu来说，切换页表，就是改一下某个CPU的值，例如x86下面的CR3寄存器）。

用户程序对内存的操作，例如malloc，只是对某个vma产生影响，不会影响页表。

但是，如果用户分配了一个内存，然后访问，又发现页表里没有相关的记录，于是产生缺页异常。

内核捕获到这个异常，然后检查vma地址是不是合法的，不合法，就是段错误。

合法，就给分配物理内存页，然后建立表项。

# 内核空间管理

除了对内存整页的使用，内核有时候，也需要像用户程序那样malloc一小块内存来用。

这个就是靠slab来实现的。

slab相当于为内核中常用的一些结构体对象建立了对象池。

例如task、mm。

slab还维护了通用的对象池，例如32字节对象池，64字节对象池。

kmalloc就是从通用对象池里取得空间的。

除了对象池，内核里还有内存池。这是2.6开始引入的 。

这里内存池的目的是：

有些对象我们不希望它分配失败，所以，我们先分配一些，放在内存池里备用。正常情况是不会去用的。

分配不到的时候，就会用内存池里的。

这就像是战略物资储备一样。

# 用户空间管理

libc对内存的分配有两种途径：

1、调整堆（heap）的大小。堆的本质也是一个vma。

2、mmap。就是得到要给新的vma。

heap相当于一个一端固定，一端可以伸缩的vma。可伸缩的那一端，是靠brk来调整的。

heap增大容易，减小却难。虽然free是可以进行减小的。

但是考虑这种情况：

```
用户连续分配了10块内存，然后释放了前面的9块，但是最后一块却没有释放，这时候，libc都不能减小堆的大小。
```



# 用户的栈

用户的栈，本质是也是一个vma。

一端固定，一端可伸（注意，不可以缩）。





# 参考文章

1、这篇文章写得系统全面，图很好。写得极好。有些话对我犹如醍醐灌顶。

http://www.cnblogs.com/zhaoyl/p/3695517.html

2、维基百科

https://www.wikiwand.com/en/Buddy_memory_allocation