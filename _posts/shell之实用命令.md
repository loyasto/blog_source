---
title: shell之实用命令
date: 2018-02-01 10:53:02
tags:
	- shell

---

###找出当前目录下最占用空间的子目录

```
du -h --max-depth=1 ./
```

###同时查找多种文件

```
find -name "Makefile" \
-o -name "*.mk" \
-o -name "*.sh" \
| xargs grep -nwr "generic\-package"
```

### 排除几种文件

```
find -type f ! -name "*.c" \
-type f ! -name "*.h" \
-type f ! -name "*.cmd" \
-type f ! -name "*.java" \
 | xargs grep -nwr "PLATFORM_VERSION_CODENAME"
```

### 查找拷贝

```

```

## stdout和stderr都重定向

```
xxx > /dev/null 2>&1
```

## 包含其他公共文件

要实现模块化编程，就需要把公共函数放在一起。

common.sh里。

```
#!/bin/sh

function mytest()
{
	echo "mytest func"
}
```

test.sh里。

```
#!/bin/sh

source ./common.sh

mytest
```



## shell里进行数学运算

```
let a="2*3"
echo $a
```

数值比较

```
"1" -lt "2"
```

```
 size_list="00400000
 00200000
 00100000
 00d00000
 00c00000
 05a00000
 17a00000"
 for sz in $size_list; do
 	sz1=`echo $((16#${sz}))`
 	flash_size=`expr $flash_size + $sz1`
 done
```

```
size_list="00400000
00200000
00100000
00d00000
00c00000
05a00000
17a00000"
for sz in $size_list; do
	sz="0x"${sz}
	sz=`printf "%d" $sz`
	flash_size=`expr $flash_size + $sz`
done
echo "FLASH:["$flash_size"],"
```



查看cpu的频率和修改。

https://blog.csdn.net/zhangyongfeiyong/article/details/52923563