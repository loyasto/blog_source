---
title: http之base64编码
date: 2018-02-27 09:02:25
tags:
	- http

---



# 为什么需要base64编码

在网络上进行数据传输的时候，要经过多个路由器，由于不同的设备对字符的处理方式不同，对于不可见字符就有可能被错误处理，导致无法正确传输。所以base64就是把所有的不可见的字符都转成可见字符，这样出错的可能性就大大降低了。

