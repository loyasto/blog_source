---
title: aosp（1）
date: 2018-11-04 11:00:02
tags:
	- Android

---



AOSP是Android Open Source Project的缩写。

1、是谷歌领导的Android开源工程。

2、AOSP可以理解为原生Android。

3、注意原生ROM和AOSP的区别是：AOSP里没有集成谷歌服务。

4、AOSP不包含任何驱动程序，需要你自己将驱动整合进去编译。



广义的原生ROM可以理解为AOSP集成了GMS（Google Mobile Service）之后得到的ROM。

GMS包括：谷歌搜索、Gmail、谷歌地图、Google+、谷歌日历、Google Play。

要想体验原生Android，有三种途径：

1、nexus和pixel手机。谷歌”亲儿子“。

2、Google Play Edition。谷歌“干儿子”。谷歌针对性地在原生ROM里支持一些厂家的旗舰机。

3、民间自己移植的。





```
sudo apt install autoconf gcc-aarch64-linux-gnu libaio-dev libbluetooth-dev libbrlapi-dev libbz2-dev libcap-dev libcap-ng-dev libcurl4-gnutls-dev libepoxy-dev libfdt-dev libgbm-dev libgles2-mesa-dev libglib2.0-dev libibverbs-dev libjpeg8-dev liblzo2-dev libncurses5-dev libnuma-dev librbd-dev librdmacm-dev libsasl2-dev libsdl1.2-dev libsdl2-dev libseccomp-dev libsnappy-dev libssh2-1-dev libtool libusb-1.0-0 libusb-1.0-0-dev libvde-dev libvdeplug-dev libvte-2.90-dev libxen-dev valgrind xfslibs-dev xutils-dev zlib1g-dev
```

这个会报错。

```
Package libvte-2.90-dev is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

E: Package 'libvte-2.90-dev' has no installation candidate
```

网上查了下，好像是libvte-2.90-dev不再提供，要用libvte-2.91-dev

改了试一下。

```
sudo apt install autoconf gcc-aarch64-linux-gnu libaio-dev libbluetooth-dev libbrlapi-dev libbz2-dev libcap-dev libcap-ng-dev libcurl4-gnutls-dev libepoxy-dev libfdt-dev libgbm-dev libgles2-mesa-dev libglib2.0-dev libibverbs-dev libjpeg8-dev liblzo2-dev libncurses5-dev libnuma-dev librbd-dev librdmacm-dev libsasl2-dev libsdl1.2-dev libsdl2-dev libseccomp-dev libsnappy-dev libssh2-1-dev libtool libusb-1.0-0 libusb-1.0-0-dev libvde-dev libvdeplug-dev libvte-2.91-dev libxen-dev valgrind xfslibs-dev xutils-dev zlib1g-dev
```

果然就可以了。

设置环境变量。

```
export PROJECT_PATH="/home/teddy/work2/aosp/qemu_android"
export VIRGLRENDERER_PATH="${PROJECT_PATH}/virglrenderer"
export QEMU_PATH="${PROJECT_PATH}/qemu"
export LINUX_PATH="${PROJECT_PATH}/linux"
export ANDROID_PATH="${PROJECT_PATH}/android"
export ANDROID_TOOLS_PATH="${PROJECT_PATH}/android-tools"
```

下载编译virglrenderer。这个是用来支持图形渲染的。

https://github.com/freedesktop/virglrenderer

不大，就1M多。

编译：

```
./autogen.sh
make
sudo make install
```

qemu的我是已经安装好的。所以先不安装。

下载内核，看教程写的是，git取最新版本。

我就直接下载最新版本的压缩包就好了。

https://github.com/torvalds/linux

现在内核源码接近200M了。

```
wget http://memcpy.io/files/2016-08-30/Kconfig -O ${LINUX_PATH}/.config
make oldconfig
make -j
```

这种配置方式，对我当前已经行不通了。

但是作者给出一些配置参考。我只要保证对应的配置是对的就好了。

当前内核版本是4.19的。



如果安装了多个版本的java。用这个命令来选择。

```
teddy@teddy-ubuntu:~/work2/aosp/qemu_android/android$ sudo update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
* 0            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      auto mode
  1            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode
  2            /usr/lib/jvm/jdk1.6.0_45/bin/java                300       manual mode

Press <enter> to keep the current choice[*], or type selection number: 0
```

我选择为0 。

javac的同样要选。

```
 sudo update-alternatives --config javac
```



先不管前面的了。现在直接下载aosp的代码。

```
teddy@teddy-ubuntu:~/work2/aosp/qemu_android/android$ git config --global user.name "teddyxiong53"
teddy@teddy-ubuntu:~/work2/aosp/qemu_android/android$ git config --global user.email "teddyxiong53@gmail.com"
teddy@teddy-ubuntu:~/work2/aosp/qemu_android/android$ 
teddy@teddy-ubuntu:~/work2/aosp/qemu_android/android$ repo init -u https://android.googlesource.com/platform/manifest
```

昨晚下载了一晚上，早上起来看，发现repo sync出错退出了。真的坑。

用这个脚本，如果repo sync出错退出，重新执行。

```
#!/bin/sh

repo sync
while [ $? -ne 0  ]; do
    repo sync
done
```

晚上我的vps下载速度可以到2M左右。白天只有300K左右。

放着慢慢下吧。



# 参考资料

1、Building Android for Qemu: A Step-by-Step Guide

https://www.collabora.com/news-and-blog/blog/2016/09/02/building-android-for-qemu-a-step-by-step-guide/

2、Android-x86虚拟机安装配置全攻略

https://cleanli.github.io/cleanhome/posts/2017-06-07/Android_x86_debug_config.html

3、源码编译运行android emulator

https://www.cnblogs.com/wxishang1991/p/5680297.html

4、

https://source.android.com/source/downloading