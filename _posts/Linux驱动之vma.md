---
title: Linux驱动之vma
date: 2018-03-12 20:26:34
tags:
	- Linux

---



一直不太明白vm_area_struct这个结构体的用途是什么。现在系统了解一下。

#定义

vma是Virtual Memory Area，虚拟内存区域的缩写。

一个vma就是一块连续的线性地址空间的抽象。每一个vma都由一个vm_area_struct结构体来描述。

是虚拟内存管理的最基本单位。描述的是一段连续的，具有相同访问属性的虚拟内存空间。大小是物理页的整数倍。





一个进程的多个vma区域要按照一定的形式组织在一起。

一个进程使用到大量虚拟内存空间不连续，而且各个部分的访问属性也不同（例如，text段只读，data读写等）。

所以，一个进程的虚拟内存空间需要多个vm_area_struct来描述。



这些vma都包含在mm_struct结构体里。

vma可以用两种方式进行组织：

1、链表方式。对应mm_struct里的mmap链表头。用来遍历。

2、红黑树方式。对应mm_struct中的mm_rb根节点。用来查找。是为了提高效率。



下面以file的address_space为例，看看跟vma是如何关联起来的。

```
struct address_space {
	struct inode		*host;
	struct rb_root		i_mmap;	
```

file和inode结构体都包含了address_space指针。

file是一个磁盘文件跟进程相关的部分。

inode则是跟进程无关的。



