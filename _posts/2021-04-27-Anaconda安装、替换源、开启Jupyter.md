---
layout: post
title: Anaconda安装、替换国内源、创建新环境及Jupyter操作
categories: [python]
description: Anaconda安装及安装后的操作
keywords: Anaconda, Jupyter, 替换源
---

## 安装 Anaconda

[官方下载地址](https://www.anaconda.com/products/individual)

[清华镜像下载地址](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)

下载完成后双击安装包进行安装。

![](/images/posts/python/Anaconda3-1.jpg)

一直下一步，并修改以下部分。

![](/images/posts/python/Anaconda3-2.jpg)

![](/images/posts/python/Anaconda3-3.jpg)

![](/images/posts/python/Anaconda3-4.jpg)

## 修改国内镜像源

打开 Anaconda Prompt，依次输入以下命令：
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
```

配置完成后可在命令行输入以下命令查看 channel URLs 是否添加成功：

```
conda info
```
or
```
conda config --show channels
```

## 创建新的 Python 环境

打开 Anaconda Prompt，依次输入以下命令：

```
conda create --name name python=X.X.X

activate name
```

其中 name 是创建新环境的名字，X.X.X 表示 python 的版本，activate 表示切换到某个环境。

以下命令可查看已有的所有环境：
```
conda info -e
```

## 包管理（安装、更新、卸载）

conda 的包管理功能和 pip 是一样的，当然选择 pip 来安装包也是没问题的。

### 搜索包
```
conda search XXX
```

### 安装包

例如安装 matplotlib 包：
```
conda install matplotlib
```
or
```
conda install matplotlib==X.X.X
```

### 卸载包

例如删除 matplotlib 包：
```
conda remove matplotlib
```

### 卸载包

例如更新 matplotlib 包：
```
conda update matplotlib
```

### 查看已安装的包的列表

```
conda list
```

## 安装 Jupyter

命令行安装：
```
conda install jupyter notebook
```

打开 Jupyter notebook
```
cd 指定文件夹

jupyter notebook
```

## 卸载 Anaconda

[这是官方说明的卸载方法](https://docs.anaconda.com/anaconda/install/uninstall/)

### 方案一

通常卸载软件，直接运行 uninstall 就可以了，对于 anaconda 也一样，可以直接运行安装目录下的 Uninstall-Anaconda3.exe 即可，但是这样卸载并没有完全卸载。如果需要完全卸载请参考**方案二**。

### 方案二

在进行方案二卸载 Anaconda 时，需先确保**未进行方案一**的卸载流程。

1.打开 Anaconda Prompt，安装 Anaconda-Clean package

```
conda install anaconda-clean
```

2.输入如下命令卸载

```
anaconda-clean --yes
```

3.按照方案一的方式再卸载一遍