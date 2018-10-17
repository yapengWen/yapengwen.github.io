---
title: Anaconda的使用及配置方法
date: 2017-07-17 09:27:38
tags: Anaconda
categories: Python
---

俗话说‘人生苦短，我有Python’，但是如果初学python的过程中碰到包和Python版本的问题估计会让你再苦一会，我在学习Python的爬虫框架中看到看到了anaconda的介绍，简直是相见恨晚啊，我觉的每个Python的学习网站上首先都应该使用anaconda来进行教程，因为在实践的过程中光环境的各种报错就能消磨掉你所有的学习兴趣！

- 下面简单的介绍下anaconda，它是将Python版本和许多常用的package打包直接来使用的Python发行版，支持Linux、mac、windows系统，并有一个conda强大的执行工具。使用起来绝对会让你舒服
- 
##### Anaconda的安装

不同的操作系统都是直接的在官网中下载安装包进行下载，选择你最经常使用的Python版本进行安装，下载完之后，尽量的按照anaconda默认的行为安装，安装时会自动的吧bin目录加入到环境变量path中去。

- 安装成功就可以通过：conda --version命令进行检验是否安装成功。
- 还可以通过python --version 命令查看发行版默认的Python版本。
- 
###### conda的常用命令操作

conda管理工具可以同时安装不同版本的python，并且自由的进行切换，经常使用的有以下的命令：


```
# 创建一个名为python34的环境，指定Python版本是3.4（不用管是3.4.x，conda会为我们自动寻找3.4.x中的最新版本）
conda create --name python34 python=3.4

# 安装好后，使用activate激活某个环境
activate python34 # for Windows
source activate python34 # for Linux & Mac
# 激活后，会发现terminal输入的地方多了python34的字样，实际上，此时系统做的事情就是把默认2.7环境从PATH中去除，再把3.4对应的命令加入PATH

# 此时，再次输入
python --version
# 可以得到`Python 3.4.5 :: Anaconda 4.1.1 (64-bit)`，即系统已经切换到了3.4的环境

# 如果想返回默认的python 2.7环境，运行
deactivate python34 # for Windows
source deactivate python34 # for Linux & Mac

# 删除一个已有的环境
conda remove --name python34 --all
```
###### 使用conda管理包


```
# 安装scipy
conda install scipy
# conda会从从远程搜索scipy的相关信息和依赖项目，对于python 3.4，conda会同时安装numpy和mkl（运算加速的库）

# 查看已经安装的packages
conda list
# 最新版的conda是从site-packages文件夹中搜索已经安装的包，不依赖于pip，因此可以显示出通过各种方式安装的包

# 查看当前环境下已安装的包
conda list

# 查看某个指定环境的已安装包
conda list -n python34

# 查找package信息
conda search numpy

# 安装package
conda install -n python34 numpy
# 如果不用-n指定环境名称，则被安装在当前活跃环境
# 也可以通过-c指定通过某个channel安装

# 更新package
conda update -n python34 numpy

# 删除package
conda remove -n python34 numpy

# 更新conda，保持conda最新
conda update conda

# 更新anaconda
conda update anaconda

# 更新python
conda update python
# 假设当前环境是python 3.4, conda会将python升级为3.4.x系列的当前最新版本
```

注意：在以上的使用过程中你会发现使用conda的下载速度非常的慢，因为使用的是国外的服务器，所以这里要设置为国内的镜像。使用下面的配置命令即可：

```
# 添加Anaconda的TUNA镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
# TUNA的help中镜像地址加有引号，需要去掉

# 设置搜索时显示通道地址
conda config --set show_channel_urls yes
```

本文转自[http://blog.csdn.net/leoe_/article/details/70544190](http://blog.csdn.net/leoe_/article/details/70544190)