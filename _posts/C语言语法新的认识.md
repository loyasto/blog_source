---
title: C语言语法新的认识
date: 2017-11-18 22:15:20
tags:
	- C语言
	- 语法

---



在阅读其他人的代码过程中，有看到一些新奇的用法，是我之前没有想到的，把这些点总结下来，理解提高。

1、用变量做数字的长度，用在定义变量的地方。

```
int n = 10;
int a[n];
```

我在Linux下，用gcc验证了，是可以的。网上查了下，vc6是不支持的。

gcc的语法对于数组下标做了不少奇怪的扩展。数组长度可以是0，可以是负数。如果要跨平台的软件，最好别这么用吧。好像也没有带来太大的好处。编译器厂家大多不跟进。



