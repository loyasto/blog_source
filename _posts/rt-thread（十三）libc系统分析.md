---
title: rt-thread（十三）libc系统分析
date: 2018-02-06 10:40:48
tags:
	- rt-thread

---



rt-thread的libc系统初始化是libc_system_init。

这个是作为一个component初始化的。

在libc初始化之后，才能使用printf进行打印。



libc_system_init

```
1、rt_console_get_device。
2、libc_stdio_set_console。这个是在components/libc/comilers/newlib里。
```



libc_stdio_set_console函数

```
1、fopen打开/dev/console设备。
2、setvbuf关闭缓冲。
3、把std_console这个fp指针赋值给_GLOBAL_REENT（这个全局变量是newlib特有的）的stdin、stdout/stderr。
4、putenv设置环境变量。PATH=/bin, HOME=/home
5、pthread_system_init。
	key、mq、sem的初始化，因为这3个东西需要全局资源的支持，所以要提前准备一下。这样后面用pthread就不会出错了。
```

