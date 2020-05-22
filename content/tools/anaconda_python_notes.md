---
title: "Anaconda notes"
date: 2019-11-13 01:01
tag: python
---

Anaconda 是一个用于科学计算的 Python 发行版，支持 Linux, Mac, Windows, 包含了众多流行的科学计算、数据分析的 Python 包。

[TOC]

## 替换源

推荐[清华大学的 Anaconda 源](https://mirror.tuna.tsinghua.edu.cn/help/anaconda/)：

直接编辑 `~/.condarc` 文件：

```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

`pip` 改用清华的源：

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pandas
```

## 常用操作

### 创建一个新的环境

```
conda create -n py37 python=3.7

source activate py37  # 激活环境
source deactivate  # 退出环境
```

### 复制一个环境

```
conda create -n py37_backup --clone py37
```

### 导出/导入环境

```
conda env list  # 列出所有环境

source activate py37  # 进入要导出的环境
conda env export --file py37.yml

# 到另外一台机器上，利用 yml 文件导入环境
conda env create -f py37.yml
```

### 删除环境

```
# 删除指定环境的 package
conda remove -n py37 <package_name>

# 完全删除一个环境
conda remove -n py37 --all
```

### 安装包

```
# 指定环境安装包
conda install -n py37 <package_name>

# 指定 channel
conda install -n py37 -c <channel_name> <package_name>
```
