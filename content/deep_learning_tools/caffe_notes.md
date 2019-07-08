---
title: "Caffe_notes"
date: 2019-07-09 01:01
---

[TOC]

> [Caffe](http://caffe.berkeleyvision.org/)，全称Convolutional Architecture for Fast Feature Embedding，是一个兼具表达性、速度和思维模块化的深度学习框架。由伯克利人工智能研究小组和伯克利视觉和学习中心开发。虽然其内核是用C++编写的，但Caffe有Python和Matlab 相关接口。Caffe支持多种类型的深度学习架构，面向图像分类和图像分割，还支持CNN、RCNN、LSTM和全连接神经网络设计。Caffe支持基于GPU和CPU的加速计算内核库，如NVIDIA cuDNN和Intel MKL。

---

## Caffe 依赖库

### ProtoBuffer

Google 开发的一种内存与非易失存储介质（如硬盘文件）交换的协议接口。

Caffe 用 PB 来保存 *权值* 和 *模型参数* 。

使用统一的参数描述文件 (.proto)，根据 .proto 文件生成协议细节代码。

### Boost

强大的开源 C++ 库，C++ 标准库的后备。

Caffe 使用了 Boost 的智能指针，利用 Boost Python 实现了 `pycaffe` 。

### GFLAGS

Google 的开源命令行参数解析库。

### GLOG

Google 的开源日志库。

### BLAS

CPU 端矩阵计算库。

### HDF5

HDF, Hierarchical Data File，一种数据格式。实现高效的存储、分发。

### OpenCV

图像读写、预处理操作。Caffe 实际上用到 OpenCV 的部分很少。

### LMDB 和 LevelDB

LMDB -- Lightning Memory-Mapped Database Manager，在 Caffe 中提供数据管理功能，将图像、二进制数据等统一用 key-value 形式存储，便于 `DataLayer` 读取。

LevelDB 是老版本 Caffe 使用的数据库，目前大部分例子使用 LMDB。

### Snappy

一个用来压缩和解压缩的 C++ 库，比 zlib 快，但是文件要更大。

---

## Caffe 代码阅读建议

- 先看 `src/caffe/proto/caffe.proto`，了解基本数据结构内存对象与磁盘文件的一一映射关系；
- 看头文件，了解每个模块大概的作用；
- 有针对性地去看 `cpp` 和 `cu` 文件；
- 一般按需求去派生新的类即可，如继承 `ConvolutionLayer` 去实现自己的卷积层
- 根据 `tools` 下面的工具去自己修改、编写新工具

利用 grep 来迅速定位代码：

```
grep -n -H -R "xxx" *

# -n 显示行号
# -H 显示文件名
# -R 递归查找子目录
```

---

## Caffe 前馈神经网络例子

- 输入层为二维图像数据，前几层均为卷积层，提取特征；最后两层为全连接层，类似于多层感知机，用于对前面提供的特征进行分类。
- 卷积层和全连接层统称为权值层，它们都有需要学习出的参数（权值）。

### 卷积层

- 多个通道，每个通道按照二维卷积方式计算；
- 多个通道与多个卷积核分别进行卷积，得到多通道输出，需要“合并”为一个通道；
- 假设卷积层有 `L` 个输出通道和 `K` 个输入通道，需要 `L*K` 个卷积核实现通道数转换；
- 假设卷积核的大小为 `I*J`，每个输出通道的 feature map （特征图）大小均为 `M*N`，则该层每个样本做一次前向传播时的计算量为： `Calculations = I * J * M * N * K * L`
- 在 mnist/lenet_train_val.prototxt 中找到卷积核参数描述：

```
convolution_param {
    num_output: 50  // 对应上面的 L
    kernel_size: 5  // 对应上面的 I J
    stride: 1  // 滑动窗口每次移动的步长
    ......
}
```

- 当前层的学习参数数量： `Params = I * J * K * L`
- CPR (Calculations to Parameters Ratio): `CPR = M * N`
- 卷积层的特征图的尺寸越大，CPR 越大，参数重复利用率越高

### 全连接层

- CNN 之前，深度学习网络计算模型都是全连接形式的 DNN，每个节点与相邻层所有节点都有连接关系；
- 全连接层主要计算为“矩阵与向量乘积”： $y = W * x$，这里 x 为输入向量，y 为输出向量，W 为权值矩阵；
- 全连接层中，直接将前一层输出展开为一维向量（不考虑 batch 的维度）：

```
inner_product_param {
    num_output: 500  // 输出向量维数
    ......
}
```

- 设 W 矩阵大小为 $V * D$ ，全连接层计算量与参数量都是 O($V * D$)，因此全连接层的 CPR 与输入、输出维度无关；
- 上面考虑的是全连接层处理单个样本的情况。如果将一批样本（B个）逐列拼接成输入矩阵 X，此时参数量不变，但计算量提高了B倍，权值矩阵在多样本之间实现了重用；
- Caffe 中全连接层的实现可以查找 `InnerProductLayer`；
- 卷积层相比全连接层，实现了权值共享，这是降低参数量的重要举措，同时卷积层的局部连接特性也大幅减少了参数量。
- 这对这些特点，大部分 CNN 网络中前几层卷积层参数量占比小，计算量占比大；后几层全连接层参数量占比大，计算量占比小。因此，加速计算优化时，重点在于卷积层；参数优化、权值裁剪时，重点在于全连接层。

### 激活函数

- Activation function 或 Squashing function；
- 非线性单元，多层神经网络如果不引入非线性单元，总可以化简为单层网络；
- 常见的激活函数有 sigmoid, tanh, ReLU 等；
- DNN 训练会遇到梯度消失问题（Gradient Vanishing Problem），这在使用 sigmoid, tanh 等饱和激活函数时尤为严重；
- 神经网络进行误差反向传播时，**各层都要乘以激活函数的一阶导数**，梯度每传递一层都要衰减一次，网络层数较多时，梯度会不停衰减直至消失；这使得训练时收敛速度非常慢；
- 使用 ReLU 这类非饱和激活函数训练收敛速度会快很多；
- Caffe 中激活函数相关的 `Layer` 在 `include/caffe/neural_layers.hpp` 中，可以称为**非线性层**；
- 非线性层的共同特点就是对前一层 `blob` 中的数值逐一进行非线性变换，再放回原 `blob` 中。

---

## Caffe 中的数据结构

（TODO: ...）



