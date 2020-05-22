---
title: "Dive Into Deep Learning notes(1) -- 卷积神经网络1"
date: 2019-10-08 21:34
collection: d2lzh
---

[TOC]

[动手学深度学习](http://zh.d2l.ai/index.html)

---

李沐大神的基于 Mxnet 的 deep learning 入门课程。

# ch1 to ch4

看了前三章，套路还是按 线性回归 （均方损失）-> softmax（交叉熵损失）-> 多层感知机 -> nn 这个顺序来介绍。

第三章最后的 kaggle 实战比较有意思，这其实是一个经典的 feature engineering 实践，可能模型不需要怎么设计，特征提好了就有好成绩。看了下讨论区，一些需要注意的点有：

1.题目里评价标准是 log rmse，需要取对数，但是取对数要注意预测值不能为负，数值稳定性方面有些坑
2.特征挑选，这个单独都可以开门课了，比如删去 NA 过多的特征，需要熟练掌握数据处理工具（如pandas）
3.k-fold 交叉验证，数据量不大的时候可以快速验证模型过/欠拟合情况
4.改完模型先看看 loss 曲线情况，如果不平滑（抖动大）说明模型不合适

第四章讲了下 mxnet gluon 中深度学习计算相关的内容，如模型的定义/参数的存储、初始化等。

---

# ch5 卷积神经网络(1)

## 卷积和相关

![二维互相关运算](/wiki/attach/images/d2lzh_notes/figure_5.1.png)

二维卷积和相关(correlation)，其实常见的是相关操作，按卷积的严格定义是把卷积核旋转 180 度后做相关操作。不过对于 CNN 中，这两种操作没区别：

> 实际上，卷积运算与互相关运算类似。**为了得到卷积运算的输出，我们只需将核数组左右翻转并上下翻转，再与输入数组做互相关运算。**可见，卷积运算和互相关运算虽然类似，但如果它们使用相同的核数组，对于同一个输入，输出往往并不相同。
> 那么，你也许会好奇卷积层为何能使用互相关运算替代卷积运算。其实，**在深度学习中核数组都是学出来的：卷积层无论使用互相关运算或卷积运算都不影响模型预测时的输出**。为了解释这一点，假设卷积层使用互相关运算学出图5.1中的核数组。设其他条件不变，使用卷积运算学出的核数组即图5.1中的核数组按上下、左右翻转。也就是说，图5.1中的输入与学出的已翻转的核数组再做卷积运算时，依然得到图5.1中的输出。为了与大多数深度学习文献一致，**如无特别说明，本书中提到的卷积运算均指互相关运算**。

## 特征图和感受野

二维卷积层输出的二维数组可以看作是输入在空间维度（宽和高）上某一级的表征，也叫特征图（feature map）。影响元素xx的前向计算的所有可能输入区域（可能大于输入的实际尺寸）叫做xx的感受野（receptive field）。

> 我们可以通过更深的卷积神经网络使特征图中单个元素的感受野变得更加广阔，从而捕捉输入上更大尺寸的特征。

## padding 和 stride

填充（padding）是指在输入高和宽的两侧填充元素（通常是0元素）。

卷积窗口从输入数组的最左上方开始，按从左往右、从上往下的顺序，依次在输入数组上滑动。我们将每次滑动的行数和列数称为步幅（stride）。

![padding and stride](/wiki/attach/images/d2lzh_notes/padding_stride.png)

## 多输入通道和多输出通道

多输入通道：当输入数据含多个通道时，我们需要**构造一个输入通道数与输入数据的通道数相同的卷积核**，从而能够与含多通道的输入数据做互相关运算。

![](/wiki/attach/images/d2lzh_notes/figure_5.4.png)

多输出通道：当输入通道有多个时，因为我们对各个通道的结果做了累加，所以不论输入通道数是多少，输出通道数总是为1。设卷积核输入通道数和输出通道数分别为c_i和c_o，高和宽分别为k_h和k_w。如果希望得到含多个通道的输出，我们可以为每个输出通道分别创建形状为 [c_i,  k_h, k_w] 的核数组。将它们在输出通道维上连结，卷积核的形状即 [c_o, c_i, k_h, k_w]。在做互相关运算时，每个输出通道上的结果由卷积核在该输出通道上的核数组与整个输入数组计算而来。

## 1x1 卷积

最后我们讨论**卷积窗口形状为1x1的多通道卷积层**。我们通常称之为1×1卷积层，并将其中的卷积运算称为1×1卷积。因为使用了最小窗口，1×1卷积失去了卷积层可以识别高和宽维度上相邻元素构成的模式的功能。实际上，1×1卷积的主要计算发生在**通道维**上。图5.5展示了使用输入通道数为3、输出通道数为2的1×1卷积核的互相关计算。值得注意的是，**输入和输出具有相同的高和宽**。输出中的每个元素来自输入中在高和宽上相同位置的元素在不同通道之间的按权重累加。**假设我们将通道维当作特征维，将高和宽维度上的元素当成数据样本，那么1×1卷积层的作用与全连接层等价**。

![](/wiki/attach/images/d2lzh_notes/figure_5.5.png)

## 池化层

池化（pooling）层的提出是为了缓解卷积层对位置的过度敏感性。

![](/wiki/attach/images/d2lzh_notes/figure_5.6.png)

池化的实际意义是**采样**。

常用的有最大池化和平均池化。

“最小池化”有没有意义？某种程度上最小、最大是对称概念，二者等价；但常见应用里像素值 0~255，最小池化会导致特征大部分为0；还有就是激活函数的影响，如果用 relu，最小池化肯定会丢掉很多信息。

和卷积类似，池化层也可以设置 padding 和 stride

在处理多通道输入数据时，池化层对每个输入通道分别池化，而不是像卷积层那样将各通道的输入按通道相加。这意味着**池化层的输出通道数与输入通道数相等**。

## LeNet

使用含单隐藏层的多层感知机模型对图像进行分类，每张图像高和宽均是28像素。我们将图像中的像素逐行展开，得到长度为784的向量，并输入进全连接层中。然而，这种分类方法有一定的局限性：

1. 图像在同一列邻近的像素在这个向量中可能相距较远。它们构成的**模式**可能难以被模型识别。
2. 对于大尺寸的输入图像，**使用全连接层容易造成模型过大**。假设输入是高和宽均为1000x1000像素的彩色照片（含3个通道）。即使全连接层输出个数仍是256，该层权重参数的形状是(3000000×256, 3000000×256)：它占用了大约3GB的内存或显存。这带来过复杂的模型和过高的存储开销。

卷积层尝试解决这两个问题。一方面，卷积层**保留输入形状**，使图像的像素在高和宽两个方向上的相关性均可能被有效识别；另一方面，卷积层通过**滑动窗口**将同一卷积核与不同位置的输入重复计算，从而避免参数尺寸过大。

用全连接层直接进行图像分类，忽略了二维空间信息。

卷积神经网络就是含卷积层的网络。本节里我们将介绍一个早期用来识别手写数字图像的卷积神经网络：LeNet 。这个名字来源于LeNet论文的第一作者Yann LeCun。LeNet展示了通过梯度下降训练卷积神经网络可以达到手写数字识别在当时最先进的结果。这个奠基性的工作第一次将卷积神经网络推上舞台，为世人所知。

LeNet分为**卷积层块**和**全连接层块**两个部分。

卷积层块里的基本单位是**卷积层后接最大池化层**：卷积层用来识别图像里的空间模式，如线条和物体局部，之后的**最大池化层则用来降低卷积层对位置的敏感性**。卷积层块由两个这样的基本单位重复堆叠构成。在卷积层块中，每个卷积层都使用5×5的窗口，并在输出上使用sigmoid激活函数。第一个卷积层输出通道数为6，第二个卷积层输出通道数则增加到16。**这是因为第二个卷积层比第一个卷积层的输入的高和宽要小，所以增加输出通道使两个卷积层的参数尺寸类似**。卷积层块的两个最大池化层的窗口形状均为2×2，且步幅为2。**由于池化窗口与步幅形状相同，池化窗口在输入上每次滑动所覆盖的区域互不重叠**。

卷积层块的输出形状为(批量大小, 通道, 高, 宽)。当卷积层块的输出传入全连接层块时，全连接层块会将小批量中每个样本变平（flatten）。也就是说，全连接层的输入形状将变成二维，其中第一维是小批量中的样本，第二维是每个样本变平后的向量表示，且向量长度为通道、高和宽的乘积。全连接层块含3个全连接层。它们的输出个数分别是120、84和10，其中10为输出的类别个数。

```python
import d2lzh as d2l
import mxnet as mx
from mxnet import autograd, gluon, init, nd
from mxnet.gluon import loss as gloss, nn
import time

net = nn.Sequential()
net.add(nn.Conv2D(channels=6, kernel_size=5, activation='sigmoid'),
        nn.MaxPool2D(pool_size=2, strides=2),
        nn.Conv2D(channels=16, kernel_size=5, activation='sigmoid'),
        nn.MaxPool2D(pool_size=2, strides=2),
        # Dense会默认将(批量大小, 通道, 高, 宽)形状的输入转换成
        # (批量大小, 通道 * 高 * 宽)形状的输入
        nn.Dense(120, activation='sigmoid'),
        nn.Dense(84, activation='sigmoid'),
        nn.Dense(10))
```

* 卷积神经网络就是含卷积层的网络。
* LeNet交替使用卷积层和最大池化层后接全连接层来进行图像分类。

## AlexNet

在LeNet提出后的将近20年里，神经网络一度被其他机器学习方法超越，如支持向量机。虽然LeNet可以在早期的小数据集上取得好的成绩，但是在更大的真实数据集上的表现并不尽如人意。一方面，神经网络计算复杂。虽然20世纪90年代也有过一些针对神经网络的加速硬件，但并没有像之后GPU那样大量普及。因此，训练一个多通道、多层和有大量参数的卷积神经网络在当年很难完成。另一方面，当年研究者还没有大量深入研究参数初始化和非凸优化算法等诸多领域，导致复杂的神经网络的训练通常较困难。

我们在上一节看到，神经网络可以直接基于图像的原始像素进行分类。这种称为端到端（end-to-end）的方法节省了很多中间步骤。然而，在很长一段时间里更流行的是研究者通过勤劳与智慧所设计并生成的手工特征。这类图像分类研究的主要流程是：

1. 获取图像数据集；
2. 使用已有的特征提取函数生成图像的特征；
3. 使用机器学习模型对图像的特征分类。

当时认为的机器学习部分仅限最后这一步。如果那时候跟机器学习研究者交谈，他们会认为机器学习既重要又优美。优雅的定理证明了许多分类器的性质。机器学习领域生机勃勃、严谨而且极其有用。然而，如果跟计算机视觉研究者交谈，则是另外一幅景象。他们会告诉你图像识别里“不可告人”的现实是：计算机视觉流程中真正重要的是数据和特征。也就是说，使用较干净的数据集和较有效的特征甚至比机器学习模型的选择对图像分类结果的影响更大。

2012年，AlexNet横空出世。这个模型的名字来源于论文第一作者的姓名Alex Krizhevsky。AlexNet使用了8层卷积神经网络，并以很大的优势赢得了ImageNet 2012图像识别挑战赛。它首次证明了学习到的特征可以超越手工设计的特征，从而一举打破计算机视觉研究的前状。

AlexNet与LeNet的设计理念非常相似，但也有显著的区别：

1. 与相对较小的LeNet相比，AlexNet包含8层变换，其中有5层卷积和2层全连接隐藏层，以及1个全连接输出层。下面我们来详细描述这些层的设计
    + AlexNet第一层中的卷积窗口形状是11×11。因为ImageNet中绝大多数图像的高和宽均比MNIST图像的高和宽大10倍以上，ImageNet图像的物体占用更多的像素，所以需要更大的卷积窗口来捕获物体。第二层中的卷积窗口形状减小到5×5，之后全采用3×3。此外，第一、第二和第五个卷积层之后都使用了窗口形状为3×3、步幅为2的最大池化层。而且，AlexNet使用的卷积通道数也大于LeNet中的卷积通道数数十倍。
    + 紧接着最后一个卷积层的是两个输出个数为4096的全连接层。这两个巨大的全连接层带来将近1 GB的模型参数。由于早期显存的限制，最早的AlexNet使用双数据流的设计使一个GPU只需要处理一半模型。幸运的是，显存在过去几年得到了长足的发展，因此通常我们不再需要这样的特别设计了
2. AlexNet将sigmoid激活函数改成了更加简单的**ReLU**激活函数。一方面，ReLU激活函数的计算更简单，例如它并没有sigmoid激活函数中的求幂运算。另一方面，ReLU激活函数在不同的参数初始化方法下使模型更容易训练。这是由于当sigmoid激活函数输出极接近0或1时，这些区域的梯度几乎为0，从而造成反向传播无法继续更新部分模型参数；而ReLU激活函数在正区间的梯度恒为1。因此，若模型参数初始化不当，sigmoid函数可能在正区间得到几乎为0的梯度，从而令模型无法得到有效训练
3. AlexNet通过**Dropout**来控制全连接层的模型复杂度。而LeNet并没有使用Dropout
4. AlexNet引入了大量的**图像增广**，如翻转、裁剪和颜色变化，从而进一步扩大数据集来缓解过拟合。

```python
import d2lzh as d2l
from mxnet import gluon, init, nd
from mxnet.gluon import data as gdata, nn
import os
import sys

net = nn.Sequential()
# 使用较大的11 x 11窗口来捕获物体。同时使用步幅4来较大幅度减小输出高和宽。这里使用的输出通
# 道数比LeNet中的也要大很多
net.add(nn.Conv2D(96, kernel_size=11, strides=4, activation='relu'),
        nn.MaxPool2D(pool_size=3, strides=2),
        # 减小卷积窗口，使用填充为2来使得输入与输出的高和宽一致，且增大输出通道数
        nn.Conv2D(256, kernel_size=5, padding=2, activation='relu'),
        nn.MaxPool2D(pool_size=3, strides=2),
        # 连续3个卷积层，且使用更小的卷积窗口。除了最后的卷积层外，进一步增大了输出通道数。
        # 前两个卷积层后不使用池化层来减小输入的高和宽
        nn.Conv2D(384, kernel_size=3, padding=1, activation='relu'),
        nn.Conv2D(384, kernel_size=3, padding=1, activation='relu'),
        nn.Conv2D(256, kernel_size=3, padding=1, activation='relu'),
        nn.MaxPool2D(pool_size=3, strides=2),
        # 这里全连接层的输出个数比LeNet中的大数倍。使用丢弃层来缓解过拟合
        nn.Dense(4096, activation="relu"), nn.Dropout(0.5),
        nn.Dense(4096, activation="relu"), nn.Dropout(0.5),
        # 输出层。由于这里使用Fashion-MNIST，所以用类别数为10，而非论文中的1000
        nn.Dense(10))
```

* AlexNet跟LeNet结构类似，但使用了**更多的卷积层**和**更大的参数空间**来拟合大规模数据集ImageNet。它是浅层神经网络和深度神经网络的分界线。
* 虽然看上去AlexNet的实现比LeNet的实现也就多了几行代码而已，但这个观念上的转变和真正优秀实验结果的产生令学术界付出了很多年。

## VGG Net

VGG net，名字来源于论文作者所在的实验室Visual Geometry Group 。VGG提出了可以通过**重复使用简单的基础块**来构建深度模型的思路。

VGG 块。VGG块的组成规律是：**连续使用数个相同的填充为1、窗口形状为3×3的卷积层后接上一个步幅为2、窗口形状为2×2的最大池化层。卷积层保持输入的高和宽不变，而池化层则对其减半。**我们使用vgg_block函数来实现这个基础的VGG块，它可以指定卷积层的数量num_convs和输出通道数num_channels。

```python
import d2lzh as d2l
from mxnet import gluon, init, nd
from mxnet.gluon import nn

def vgg_block(num_convs, num_channels):
    blk = nn.Sequential()
    for _ in range(num_convs):
        blk.add(nn.Conv2D(num_channels, kernel_size=3,
                          padding=1, activation='relu'))
    blk.add(nn.MaxPool2D(pool_size=2, strides=2))
    return blk
```

与AlexNet和LeNet一样，VGG网络由卷积层模块后接全连接层模块构成。卷积层模块串联数个vgg_block，其超参数由变量conv_arch定义。该变量指定了每个VGG块里卷积层个数和输出通道数。全连接模块则跟AlexNet中的一样。

现在我们构造一个VGG网络。它有5个卷积块，前2块使用单卷积层，而后3块使用双卷积层。第一块的输出通道是64，之后每次对输出通道数翻倍，直到变为512。因为这个网络使用了8个卷积层和3个全连接层，所以经常被称为**VGG-11**。

```python
conv_arch = ((1, 64), (1, 128), (2, 256), (2, 512), (2, 512))
def vgg(conv_arch):
    net = nn.Sequential()
    # 卷积层部分
    for (num_convs, num_channels) in conv_arch:
        net.add(vgg_block(num_convs, num_channels))
    # 全连接层部分
    net.add(nn.Dense(4096, activation='relu'), nn.Dropout(0.5),
            nn.Dense(4096, activation='relu'), nn.Dropout(0.5),
            nn.Dense(10))
    return net

net = vgg(conv_arch)
```

下面构造一个高和宽均为224的单通道数据样本来观察每一层的输出形状。

```python
net.initialize()
X = nd.random.uniform(shape=(1, 1, 224, 224))
for blk in net:
    X = blk(X)
    print(blk.name, 'output shape:\t', X.shape)

# sequential1 output shape:        (1, 64, 112, 112)
# sequential2 output shape:        (1, 128, 56, 56)
# sequential3 output shape:        (1, 256, 28, 28)
# sequential4 output shape:        (1, 512, 14, 14)
# sequential5 output shape:        (1, 512, 7, 7)
# dense0 output shape:     (1, 4096)
# dropout0 output shape:   (1, 4096)
# dense1 output shape:     (1, 4096)
# dropout1 output shape:   (1, 4096)
# dense2 output shape:     (1, 10)
```

可以看到，每次我们将输入的高和宽减半，直到最终高和宽变成7后传入全连接层。与此同时，输出通道数每次翻倍，直到变成512。因为每个卷积层的窗口大小一样，所以**每层的模型参数尺寸和计算复杂度与输入高、输入宽、输入通道数和输出通道数的乘积成正比**。VGG这种高和宽减半以及通道翻倍的设计使得多数卷积层都有相同的模型参数尺寸和计算复杂度。

VGG Net 的特点：

1. VGG全部使用3x3卷积核、2x2池化核，不断加深网络结构来提升性能。 
2. 网络变深，参数量没有增长很多，**参数量主要在3个全连接层**。 
3. 训练比较耗时的依然是**卷积层**，因**计算量比较大**。 
4. VGG有5段卷积，每段有2~3个卷积层，每段尾部用池化来缩小图片尺寸。 
5. 每段内卷积核数一样，越靠后的段卷积核数越多：64–128–256–512–512。

创新点：

- 层数更深特征图更宽。由于**卷积核专注于扩大通道数**、**池化专注于缩小宽和高**，使得模型架构上更深更宽的同时，计算量的增加放缓；
- 全连接转卷积（测试阶段）。网络测试阶段将训练阶段的三个全连接替换为三个卷积，测试重用训练时的参数，使得测试得到的全卷积网络因为没有全连接的限制，因而可以接收任意宽或高为的输入。
- VGG16相比AlexNet的一个改进是采用连续的几个3x3的卷积核代替AlexNet中的较大卷积核（11x11，5x5）。**对于给定的感受野（与输出有关的输入图片的局部大小），采用堆积的小卷积核是优于采用大的卷积核，因为多层非线性层可以增加网络深度来保证学习更复杂的模式，而且代价还比较小（参数更少）**。

关于 FC 转 1x1 Conv: 全卷积网络可以接收任意大小的图片输入，得到一个score map，对其做一个average就可以得到最终结果(one-hot label)。

## Network In Network

LeNet、AlexNet和VGG在设计上的共同之处是：先以由卷积层构成的模块充分抽取空间特征，再以由全连接层构成的模块来输出分类结果。其中，AlexNet和VGG对LeNet的改进主要在于如何对这两个模块加宽（增加通道数）和加深。

NiN 提出了另外一个思路，即**串联多个由卷积层和“全连接”层构成的小网络来构建一个深层网络**。

我们知道，卷积层的输入和输出通常是四维数组（样本，通道，高，宽），而全连接层的输入和输出则通常是二维数组（样本，特征）。**如果想在全连接层后再接上卷积层，则需要将全连接层的输出变换为四维**。回忆在“多输入通道和多输出通道”一节里介绍的**1×1卷积层。它可以看成全连接层**，其中空间维度（高和宽）上的每个元素相当于样本，通道相当于特征。因此，**NiN使用1×1卷积层来替代全连接层**，从而使空间信息能够自然传递到后面的层中去。图5.7对比了NiN同AlexNet和VGG等网络在结构上的主要区别。

![](/wiki/attach/images/d2lzh_notes/figure_5.7.png)

NiN块是NiN中的基础块。它由**一个卷积层加两个充当全连接层的1×1卷积层串联而成**。其中第一个卷积层的超参数可以自行设置，而第二和第三个卷积层的超参数一般是固定的。

```python
import d2lzh as d2l
from mxnet import gluon, init, nd
from mxnet.gluon import nn

def nin_block(num_channels, kernel_size, strides, padding):
    blk = nn.Sequential()
    blk.add(nn.Conv2D(num_channels, kernel_size,
                      strides, padding, activation='relu'),
            # 两个 1x1 卷积层，超参数固定(padding为0，stride为1)
            nn.Conv2D(num_channels, kernel_size=1, activation='relu'),
            nn.Conv2D(num_channels, kernel_size=1, activation='relu'))
    return blk
```

NiN是在AlexNet问世不久后提出的。它们的卷积层设定有类似之处。NiN使用卷积窗口形状分别为11×11、5×5和3×3的卷积层，相应的输出通道数也与AlexNet中的一致。每个NiN块后接一个步幅为2、窗口形状为3×3的最大池化层。

除使用NiN块以外，NiN还有一个设计与AlexNet显著不同：NiN去掉了AlexNet最后的3个全连接层，取而代之地，NiN**使用了输出通道数等于标签类别数的NiN块**，然后使用**全局平均池化层**对每个通道中所有元素求平均并**直接用于分类**。这里的**全局平均池化层即窗口形状等于输入空间维形状的平均池化层**。NiN的这个设计的好处是可以**显著减小模型参数尺寸**，从而缓解过拟合。然而，该设计有时会造成获得有效模型的**训练时间的增加**。

```python
net = nn.Sequential()
net.add(nin_block(96, kernel_size=11, strides=4, padding=0),
        nn.MaxPool2D(pool_size=3, strides=2),
        nin_block(256, kernel_size=5, strides=1, padding=2),
        nn.MaxPool2D(pool_size=3, strides=2),
        nin_block(384, kernel_size=3, strides=1, padding=1),
        nn.MaxPool2D(pool_size=3, strides=2), nn.Dropout(0.5),
        # 标签类别数是10
        nin_block(10, kernel_size=3, strides=1, padding=1),
        # 全局平均池化层将窗口形状自动设置成输入的高和宽
        nn.GlobalAvgPool2D(),
        # 将四维的输出转成二维的输出，其形状为(批量大小, 10)
        nn.Flatten())

X = nd.random.uniform(shape=(1, 1, 224, 224))
net.initialize()
for layer in net:
    X = layer(X)
    print(layer.name, 'output shape:\t', X.shape)

# sequential1 output shape:        (1, 96, 54, 54)
# pool0 output shape:              (1, 96, 26, 26)
# sequential2 output shape:        (1, 256, 26, 26)
# pool1 output shape:              (1, 256, 12, 12)
# sequential3 output shape:        (1, 384, 12, 12)
# pool2 output shape:              (1, 384, 5, 5)
# dropout0 output shape:           (1, 384, 5, 5)
# sequential4 output shape:        (1, 10, 5, 5)
# pool3 output shape:              (1, 10, 1, 1)
# flatten0 output shape:           (1, 10)
```

* NiN重复使用由卷积层和代替全连接层的1×1卷积层构成的NiN块来构建深层网络。
* NiN去除了容易造成过拟合的全连接输出层，而是将其替换成输出通道数等于标签类别数的NiN块和全局平均池化层。

1×1卷积核可以起到一个跨通道聚合的作用，所以进一步可以起到降维（或者升维）的作用，起到减少参数的目的

一个问题：为什么 NiN 要用两个 1x1 卷积，用一个怎么样？

- 1 个 1x1 conv 也可以实现对前一层所有 feature map 的信息进行非线性整合（引入非线性），再增加一层，应该是为了增加非线性变换的能力，从直观上来看，单个 1x1 conv 层的性能应该没有两个 1x1 conv 层的能力强（非线性拟合能力），如果继续增加 1x1 conv 层数，效果可能会更好，但是相应的计算复杂度也会提高，这里用两个应该是找一个平衡。
- 两层1x1的CNN层可以一是增加参数空间，二是多增加一层非线性转换。如果再多一层，达到效果应该会更甚。
