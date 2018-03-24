---
title: shadowsocks之Linux终端使用
date: 2018-01-22 17:46:09
tags:
	- shadowsocks
	- Linux

---



1、安装polipo。

```
sudo apt-get install polipo
```

polipo是一个轻量级的缓存web代理程序。

2、安装shadowsocks。

```
sudo apt-get install shadowsocks
```

3、配置shadowsocks。在/etc/shadowsocks/config.json里。把账号信息配置进去。

4、运行sslocal。注意运行程序不是叫shadowsocks。

发现我配置加密方式为aes-128-ctr的时候，sslocal报错，说不支持这种方式。

我把刚才安装的shadowsocks卸载掉。用pip安装。

```
sudo pip install shadowsocks
```

这样安装的shadowsocks是2.8.2的。

启动没有报错。

```
sslocal -c /etc/shadowsocks/config.json
```

一个ssr的配置文件是这样：

```
{
    "server": "64.137.198.70",
    "server_port": 5556,
    "local_address": "127.0.0.1",
    "local_port": 1080,

    "password": "doub.io/sszhfx/*doub.ws/sszhfx/*5556",
    "method": "chacha20",
    "protocol": "auth_sha1_v4",
    "protocol_param": "",
    "obfs": "tls1.2_ticket_auth",
    "obfs_param": "",
    "speed_limit_per_con": 0,
    "speed_limit_per_user": 0,

    "timeout": 120,
    "udp_timeout": 60,
    "dns_ipv6": false,
    "connect_verbose_info": 0,
    "redirect": "",
    "fast_open": false
}
```



5、现在配置polipo。

修改/etc/polipo/config文件;

```
socksParentProxy = "localhost:1080"
socksProxyType = socks5
logFile = /var/log/polipo
logLevel = 4

```

6、启动polipo服务。

```
sudo service polipo stop
sudo service polipo start
```

7、测试一下。

```
http_proxy=http://localhost:8123 curl www.google.com
```

已经可以正常访问google了。

为了方便使用，我们把http_proxy这部分加一个别名。

在`~/.bashrc`最后加上：

```
alias hp="http_proxy=http://localhost:8123"
```

然后:

```
. ~/.bashrc
```

现在

```
hp curl www.google.com
```

