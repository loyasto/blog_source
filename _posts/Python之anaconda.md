---
title: Python之anaconda
date: 2018-06-30 19:11:40
tags:
	- Python

---



anaconda是Python的一个发行版本。

包含了conda、python等180多个科学包及其依赖。

安装包比较大，有500M。

所以还有一个mini版本，叫做miniconda。

anaconda是专注于数据分析的Python发行版本。

conda是开源包和虚拟环境的管理系统。

# anaconda的好处

八个字：省时省心，分析利器。

# 使用方法

安装就是一个exe，下一步就好了。

安装完成后，需要在cmd里运行一下：

```
conda upgrade --all
```

这是为了升级所有的工具包，以避免可能发生的错误。

需要把安装路径加入到PATH里。

安装一个包：

```
conda install xxx
```

删除：

```
conda remove xxx
```

升级：

```
conda update xxx
```

查看：

```
conda list
```



## 管理虚拟环境



# 参考资料

1、致Python初学者：Anaconda入门使用指南

http://python.jobbole.com/87522/?repeat=w3tc