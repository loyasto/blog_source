---
title: Python之函数参数args和kwargs
date: 2017-09-24 21:49:30
tags:
	- python

---



在Python代码里，经常看到`*args`和`**kwargs`这种参数，具体是表示什么意思？

除了`*args`和`**kwargs`外的参数，叫做位置参数。

# *args

`*args`表示把多出来的参数，统一当成一个元组往函数里传递。

举例：

```
def func(x, *args):
    print x
    print args
func(1,2,3,4,5)
```

结果：

```
C:\Python27\python.exe D:/work/pycharm/py_test/test.py
1
(2, 3, 4, 5)
```



# **kwargs

`**kwargs`表示把多余的参数当成字典往下传。

