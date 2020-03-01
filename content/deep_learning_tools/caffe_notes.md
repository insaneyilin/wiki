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

- Caffe 中一个 CNN 模型用 Net 表示，一个 Net 对应一个 prototxt 文件
- 一个 Net 由多个 Layer 堆叠成
- Layer 由多个 Blob 组成

### Blob

一个 Blob 在内存中表示 4 维数组：(width, height, channels, num)

width 和 height 表示图像宽高，channels 表示通道数，num 表示第几张图

Blob 用于存储数据或权值、权值增量

在进行网络计算时，每一层的输入、输出都需要 Blob 对象缓冲。

Blob 对象有 `ToProto()`, `FromProto()` 方法，可以实现内存与磁盘数据的交互。（通过 BlobProto 类型读写）

通过 Blob 来看使用 PB 的好处：（为什么不在 C++ 中直接定义结构体）

－ 结构体的序列化／反序列化需要额外编程
－ 不同结构体难以做到标准统一

Blob 是一个模版类，其中封装了 SyncedMemory 类对象（存放data、diff）

### Layer

Layer 是 Caffe 的基本计算单元，至少有一个输入 Blob 和一个输出 Blob

输入是 bottom

输出是 top

一些 Layer 有 weight 和 bias

Layer 有两个运算方向：前向传播和反向传播

－ 前向传播会对输入 Blob 进行某种处理（有 weight 和 bias 的层会利用这些信息对输入进行处理），得到输出 Blob
－ 反向传播计算对输出 Blob 的 diff 进行某种处理，得到输入 Blob 的 diff（ weight 和 bias 的 diff 可能会被用到）

下面四个函数会在各个 Layer 的派生类中经常看到：

- Forward_cpu()
- Backward_cpu()
- Forward_gpu()
- Backward_gpu()

Layer 这个类是一个虚基类，大部分函数都没有实现

### Net

Net 在 Caffe 中代表一个完整的 CNN 模型，它包含若干 Layer 实例。LeNet，AlexNet 等这些网络用 prototxt 描述，在 Caffe 中就是一个 Net 对象

Net 中既有 Layer 对象，又有 Blob 对象。

Blob 对象用于存放每个 Layer 输入／输出中间结果，Layer 对象则根据 Net 描述对指定的输入 Blob 进行某些计算处理；输入 Blob 和输出 Blob 可能为同一个

注意：Blob 名字和 Layer 名字相同并不代表它们有直接关系

有两种 blob，分别为数据 blob 和权值 blob；深度学习的目的就是不断从“数据”中获取知识，存储到“模型”中，应用于后来的“数据”。

Net -> Layer -> Blob

Blob 提供了数据容器的机制，Layer 通过不通的策略使用它们，实现多元化的计算处理过程，同时提供深度学习的各种基本算法（卷积，池化，损失函数计算等）；Net 则利用 Layer 的这些机制，组合为一个完整的深度学习模型。

---

## Caffe IO 模块

之前的例子：将数据转换为 LMDB 格式，训练网络时由数据读取层（DataLayer）不断从 LMDB 读取数据，送入后续卷积、池化等层。

### 数据读取层

除了从 LMDB 、LEVELDB 等读取外，也支持从原始图像直接读取（ImageDataLayer）

- batch_size: 一个批量数据包含的图片数目
- rand_skip: 随机跳过若干图片

数据读取层代码： include/caffe/data_layers.hpp

### 数据转换器

Data Transformer

一些预处理方法：随机镜像、随机切块、去均值、灰度变换等。

其他数据层：memory_data_layer, window_data_layer, hdf5_data_layer

---

## Caffe 模型

一个完整的深度学习系统：数据 ＋ 模型

一个深度学习模型通常有三部分：

- 可学习参数（Learnable Parameter）:训练参数，神经网络权值，其数值由模型初始化参数、误差反向传播控制，一般不可人工干预
- 结构参数（Archetecture Parameter）:卷积层、全连接层、下采样层数目，卷积核数目，卷积核大小等用来描述网络结构的参数，训练前事先设定好；注意，训练阶段和预测阶段网络结构参数很可能不同
- 训练超参数（Hyper-Parameter）:用来控制网络训练收敛的参数，往往调参调的就是这个；预测网络不需要该参数

可学习参数在内存中使用 Blob 对象保持，必要时以二进制保存在磁盘上（.caffemodel），便于 finetune 、共享、与性能评估（benchmark）

结构参数通过 ProtoBuffer 文本格式描述，通过该描述文件构建 Net 对象、Layer 对象，形成 DAG（有向无环图），在 Layer 与 Layer 之间、Net 输入与输出之间均为持有数据和中间结果的 Blob 对象。

训练超参数也是通过 prototxt 保存，训练时利用该描述文件构建 Solver，该对象按照一定规则在训练网络时自动调节这些超参数

### Caffe Model Zoo

对于规模很大的模型，是否每次都需要从头训练？

Caffe Model Zoo 提供了一个分享模型的平台

---

## Caffe 前向传播计算

CNN 训练时一般包括了前向传播和反向传播两个阶段

前向传播阶段，数据从数据读取层出发，经过若干处理层，达到最后一层（损失层或特征层）

网络中的权值在 FP 过程中不发生变化，可以看作常量。

网络路径是一个有向无环图

Caffe 中 FP 通过 Net ＋ Layer 组合完成，中间结果和最终结果通过 Blob 承载。

前向传播：从输入层到输出层，一层一层利用已有权值来更新 loss 和最终输出

反向传播：从输出层到输入层，从输出层总误差开始一层一层更新权值

前向传播在训练和预测中都要用到

反向传播只在训练时用到

Loss Layer 是 CNN 的终点，接受两个 Blob 作为输入，其中一个为 CNN 的预测值，另一个是真实标签。 损失层将这两个输入进行一系列运算，计算损失函数值 L(\theta), \theta 是模型的权值，训练的目标就是得到最优权值，使得 Loss 最小。

损失函数是在前向传播计算中得到的，同时也是反向传播的起点

Caffe 中实现了多种损失层，分别用于不同场合，其中 SoftmaxWithLossLayer 实现了 Softmax ＋ 交叉熵损失函数计算过程

Caffe 训练时的的问题“为什么 Loss 总在 6.9 左右？”，因为 -ln(0.001) = 6.907755... （ImageNet-1000 分类问题初始状态为均匀分布，每个类别的分类概率为 0.001 ）。这说明没有收敛的迹象，需要调大学习率或者修改权值初始化方式

一个 Blob 中有 data 和 diff 两部分数据，数据读取层提供了 data， 损失层提供了 diff

---

## Caffe 反向传播计算

从 Loss 层向数据层方向传播 diff ，更新权值

---

## Caffe 最优化求解

Caffe 求解器的职责：

- 负责记录优化过程，创建用于学习的训练网络和用于评估学习效果的测试网络
- 调用 Forward －》 调用 Backward －》 更新权值，反复迭代优化模型
- 周期性评估测试网络
- 在优化中为模型、求解器状态打快照

Caffe 求解器每次迭代中做的事情：

- 调用 Net 的前向传播函数来计算输出和损失函数
- 调用 Net 的反向传播函数来计算梯度
- 根据求解器方法，将梯度转换为权值增量
- 根据学习率、历史权值、所用方法更新求解器状态

### Caffe 中的求解器

- SGD，随机梯度下降
- AdaDelta
- ADAGRAD，自适应梯度
- Adam
- Nesterov，加速梯度法
- RMSprop

求解器方法的重点在于最小化损失函数的全局优化问题

SGD算法，利用负梯度和权值更新历史的线性组合更新权值 W

- SGD 算法里的超参数：
- lr，学习率，负梯度的权重
- momentum，权值更新历史的权重

---

## Caffe 工具

tools/ 下有一些利用 caffe 编写的程序，如训练、预测、预处理等

---

## Caffe GPU 加速

略，CUDA ＋ cuDNN

---

## Caffe 可视化

- 原始数据可视化
- 网络可视化（利用 draw_net.py 来画）
- 网络权值可视化：通常第一个卷积层是最容易解释的，良好训练的网络权值通常表现为美观、光滑的滤波器
- 如果网络权值可视化中出现图案相关性高、缺乏结构性等情况，可能说明训练结果不好

训练脚本时这么写：

```
xxx.sh 2>&1 xxx.log &
# 2>&1 表示 stdout 和 stderr 都重定向到 xxx.log
# 最后的 & 表示在后台运行
```

可以用 tail -f xxx.log 观察 log 文件的更新

可以使用如下命令提取 log 文件中的 loss 值：

```
cat xxx.log | grep "Train net output" | awk '{print $11}'
```
