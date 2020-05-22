---
title: "Dive Into Deep Learning notes(2) -- 卷积神经网络2"
date: 2019-10-09 23:34
collection: d2lzh
---

[TOC]

[动手学深度学习](http://zh.d2l.ai/index.html)

---

# ch5 卷积神经网络(2)

## GoogleNet 含并行连接的网络

GoogleNet 中的基础结构，Inception 块：

![](/wiki/attach/images/d2lzh_notes/figure_5.8.png)

Inception块里有4条并行的线路。前3条线路使用窗口大小分别是1×1、3×3和5×5的卷积层来**抽取不同空间尺寸下的信息**，其中**中间2个线路会对输入先做1×1卷积来减少输入通道数**，以降低模型复杂度。第四条线路则使用3×3最大池化层，后接1×1卷积层来改变通道数。4条线路都**使用了合适的填充来使输入与输出的高和宽一致**。最后我们**将每条线路的输出在通道维上连结**，并输入接下来的层中去。

```python
import d2lzh as d2l
from mxnet import gluon, init, nd
from mxnet.gluon import nn

class Inception(nn.Block):
    # c1 - c4为每条线路里的层的输出通道数
    def __init__(self, c1, c2, c3, c4, **kwargs):
        super(Inception, self).__init__(**kwargs)
        # 线路1，单1 x 1卷积层
        self.p1_1 = nn.Conv2D(c1, kernel_size=1, activation='relu')
        # 线路2，1 x 1卷积层后接3 x 3卷积层
        self.p2_1 = nn.Conv2D(c2[0], kernel_size=1, activation='relu')
        self.p2_2 = nn.Conv2D(c2[1], kernel_size=3, padding=1,
                              activation='relu')
        # 线路3，1 x 1卷积层后接5 x 5卷积层
        self.p3_1 = nn.Conv2D(c3[0], kernel_size=1, activation='relu')
        self.p3_2 = nn.Conv2D(c3[1], kernel_size=5, padding=2,
                              activation='relu')
        # 线路4，3 x 3最大池化层后接1 x 1卷积层
        self.p4_1 = nn.MaxPool2D(pool_size=3, strides=1, padding=1)
        self.p4_2 = nn.Conv2D(c4, kernel_size=1, activation='relu')

    def forward(self, x):
        p1 = self.p1_1(x)
        p2 = self.p2_2(self.p2_1(x))
        p3 = self.p3_2(self.p3_1(x))
        p4 = self.p4_2(self.p4_1(x))
        # 在通道维上连结输出
        return nd.concat(p1, p2, p3, p4, dim=1)
```

GoogLeNet跟VGG一样，在主体卷积部分中使用5个模块（block），每个模块之间使用步幅为2的3×3最大池化层来减小输出高宽。

第一模块使用一个64通道的7×7卷积层。

第二模块使用2个卷积层：首先是64通道的1×1卷积层，然后是将通道增大3倍的3×3卷积层。它对应Inception块中的第二条线路。

第三模块串联2个完整的Inception块。第一个Inception块的输出通道数为 `64+128+32+32 = 256` ，其中4条线路的输出通道数比例为 `64:128:32:32 = 2:4:1:1`。其中第二、第三条线路先分别将输入通道数减小至`96/192 = 1/2` 和 `16/192=1/12` 后，再接上第二层卷积层。第二个Inception块输出通道数增至 `128+192+96+64 = 480` ，每条线路的输出通道数之比为 `128:192:96:64 = 4:6:3:2` 。其中第二、第三条线路先分别将输入通道数减小至 `128/256 = 1/2` 和 `32/256 = 1/8` 。

第四模块更加复杂。它串联了5个Inception块，其输出通道数分别是 `192+208+48+64 = 512` 、`160+224+64+64 = 512` 、`128+256+64+64 = 512` 、`112+288+64+64 = 528` 和 `256+320+128+128 = 832` 。这些线路的通道数分配和第三模块中的类似，首先含3×3卷积层的第二条线路输出最多通道，其次是仅含1×1卷积层的第一条线路，之后是含5×5卷积层的第三条线路和含3×3最大池化层的第四条线路。其中第二、第三条线路都会先按比例减小通道数。这些比例在各个Inception块中都略有不同。

第五模块有输出通道数为 `256+320+128+128 = 832` 和 `384+384+128+128 = 1024` 的两个Inception块。其中每条线路的通道数的分配思路和第三、第四模块中的一致，只是在具体数值上有所不同。需要注意的是，第五模块的后面紧跟输出层，该模块同NiN一样使用全局平均池化层来将每个通道的高和宽变成1。最后我们将输出变成二维数组后接上一个输出个数为标签类别数的全连接层。

* Inception块相当于**一个有4条线路的子网络**。它通过不同窗口形状的卷积层和最大池化层来**并行抽取信息**，并使用1×1卷积层减少通道数从而降低模型复杂度。
* GoogLeNet将多个设计精细的Inception块和其他层串联起来。其中**Inception块的通道数分配之比是在ImageNet数据集上通过大量的实验得来的**。
* GoogLeNet和它的后继者们一度是ImageNet上最高效的模型之一：在类似的测试精度下，它们的计算复杂度往往更低。

## Batch Normalization

本节我们介绍批量归一化（batch normalization）层，它能让较深的神经网络的训练变得更加容易。在 “实战Kaggle比赛：预测房价” 一节里，我们对输入数据做了标准化处理：处理后的任意一个特征在数据集中所有样本上的均值为0、标准差为1。**标准化处理输入数据使各个特征的分布相近：这往往更容易训练出有效的模型**。

通常来说，数据标准化预处理对于浅层模型就足够有效了。随着模型训练的进行，当每层中参数更新时，靠近输出层的输出较难出现剧烈变化。但**对深层神经网络来说，即使输入数据已做标准化，训练中模型参数的更新依然很容易造成靠近输出层输出的剧烈变化**。这种**计算数值的不稳定性**通常令我们难以训练出有效的深度模型。

批量归一化的提出正是为了应对深度模型训练的挑战。在模型训练时，批量归一化利用小批量上的均值和标准差，**不断调整神经网络中间输出**，从而**使整个神经网络在各层的中间输出的数值更稳定**。批量归一化和下一节将要介绍的残差网络为训练和设计深度模型提供了两类重要思路。

对全连接层和卷积层做批量归一化的方法稍有不同。下面我们将分别介绍这两种情况下的批量归一化。
通常，我们将批量归一化层置于**全连接层中的仿射变换和激活函数之间**。利用小批量样本学习 `拉伸` 和 `偏移`参数。

可学习的拉伸和偏移参数**保留了不做批量归一化的可能**，即：如果批量归一化无益，理论上，学出的模型可以不使用批量归一化。

对卷积层来说，批量归一化发生在**卷积计算之后、应用激活函数之前**。如果卷积计算输出多个通道，我们需要对这些通道的输出分别做批量归一化，且每个通道都拥有独立的拉伸和偏移参数，并均为标量。设小批量中有 `m` 个样本。在单个通道上，假设卷积计算输出的高和宽分别为 `p` 和 `q` 。我们需要对该通道中 `m×p×q` 个元素同时做批量归一化。对这些元素做标准化计算时，我们使用相同的均值和方差，即该通道中 `m×p×q` 个元素的均值和方差。

使用批量归一化训练时，我们可以 **将批量大小设得大一点，从而使批量内样本的均值和方差的计算都较为准确** 。将训练好的模型用于预测时，我们希望模型对于任意输入都有确定的输出。因此，单个样本的输出不应取决于批量归一化所需要的随机小批量中的均值和方差。一种常用的方法是 **通过移动平均估算整个训练数据集的样本均值和方差，并在预测时使用它们得到确定的输出** 。可见， **和丢弃层一样，批量归一化层在训练模式和预测模式下的计算结果也是不一样的** 。

BN 主要是让收敛变快，但对模型的预测准确率等影响不大。

## ResNet

ResNet 的出现把 dnn 做的更 deep；网络越深，越难训练：梯度在深层会变得很小（把梯度理解成信息，信息被前面的层吸收掉了，后面的层就得不到什么信息了，难训）

网络变宽，相当于用更更复杂的函数；网络变深，相当于用更多函数组合来进行拟合。

让我们先思考一个问题：对神经网络模型添加新的层，充分训练后的模型是否只可能更有效地降低训练误差？**理论上，原模型解的空间只是新模型解的空间的子空间**。也就是说，如果我们能将新添加的层训练成**恒等映射f(x)=x**，新模型和原模型将同样有效。由于新模型可能得出更优的解来拟合训练数据集，因此添加层似乎更容易降低训练误差。然而**在实践中，添加过多的层后训练误差往往不降反升**。即使利用批量归一化带来的数值稳定性使训练深层模型更加容易，该问题仍然存在。针对这一问题，何恺明等人提出了残差网络（ResNet）。它在2015年的ImageNet图像识别挑战赛夺魁，并深刻影响了后来的深度神经网络的设计。

残差块。让我们聚焦于神经网络局部。如图5.9所示，设输入为x。假设我们希望学出的理想映射为f(x)，从而作为图5.9上方激活函数的输入。左图虚线框中的部分需要 **直接拟合出该映射f(x)**，而右图虚线框中的部分则需要 **拟合出有关恒等映射的残差映射f(x)−x** 。 **残差映射在实际中往往更容易优化** 。以本节开头提到的恒等映射作为我们希望学出的理想映射f(x)。我们只需将图5.9中右图虚线框内上方的加权运算（如仿射）的权重和偏差参数学成0，那么f(x)即为恒等映射。实际中，当理想映射f(x)极接近于恒等映射时，**残差映射也易于捕捉恒等映射的细微波动**。图5.9右图也是ResNet的基础块，即残差块（residual block）。在残差块中，输入可通过跨层的数据线路更快地向前传播。

![](/wiki/attach/images/d2lzh_notes/figure_5.9.png)

ResNet沿用了VGG全3×3卷积层的设计。残差块里首先有2个有相同输出通道数的3×3卷积层。每个卷积层后接一个批量归一化层和ReLU激活函数。然后我们将输入跳过这两个卷积运算后直接加在最后的ReLU激活函数前。这样的设计要求两个卷积层的输出与输入形状一样，从而可以相加。**如果想改变通道数，就需要引入一个额外的1×1卷积层来将输入变换成需要的形状后再做相加运算**。

```python
import d2lzh as d2l
from mxnet import gluon, init, nd
from mxnet.gluon import nn

class Residual(nn.Block):  # 本类已保存在d2lzh包中方便以后使用
    def __init__(self, num_channels, use_1x1conv=False, strides=1, **kwargs):
        super(Residual, self).__init__(**kwargs)
        self.conv1 = nn.Conv2D(num_channels, kernel_size=3, padding=1,
                               strides=strides)
        self.conv2 = nn.Conv2D(num_channels, kernel_size=3, padding=1)
        if use_1x1conv:
            self.conv3 = nn.Conv2D(num_channels, kernel_size=1,
                                   strides=strides)
        else:
            self.conv3 = None
        self.bn1 = nn.BatchNorm()
        self.bn2 = nn.BatchNorm()

    def forward(self, X):
        Y = nd.relu(self.bn1(self.conv1(X)))
        Y = self.bn2(self.conv2(Y))
        if self.conv3:
            X = self.conv3(X)
        return nd.relu(Y + X)
```

下面我们来查看输入和输出形状一致的情况：

```python
blk = Residual(3)
blk.initialize()
X = nd.random.uniform(shape=(4, 3, 6, 6))
blk(X).shape
# (4, 3, 6, 6)
```

我们也可以在增加输出通道数的同时减半输出的高和宽：

```python
blk = Residual(6, use_1x1conv=True, strides=2)
blk.initialize()
blk(X).shape
# (4, 6, 3, 3)
```

ResNet的前两层跟之前介绍的GoogLeNet中的一样：在输出通道数为64、步幅为2的7×7卷积层后接步幅为2的3×3的最大池化层。不同之处在于ResNet每个卷积层后增加的批量归一化层。

GoogLeNet在后面接了4个由Inception块组成的模块。ResNet则使用4个由残差块组成的模块，每个模块使用若干个同样输出通道数的残差块。第一个模块的通道数同输入通道数一致。由于之前已经使用了步幅为2的最大池化层，所以无须减小高和宽。之后的每个模块在第一个残差块里将上一个模块的通道数翻倍，并将高和宽减半。

虽然ResNet的主体架构跟GoogLeNet的类似，但ResNet结构更简单，修改也更方便。这些因素都导致了ResNet迅速被广泛使用。（ResNet 胜在简单，GoogleNet 中设计复杂，“看不懂”）

* 残差块通过跨层的数据通道从而能够训练出有效的深度神经网络。

在ResNet的后续版本(ResNet v2)里，作者将残差块里的“卷积、批量归一化和激活”结构改成了“批量归一化、激活和卷积”，即 "full pre-activation"，将BN和Relu都放在了residual通路的一开始，使得梯度更好地反向传播，易于优化。

## DenseNet

ResNet中的跨层连接设计引申出了数个后续工作。本节我们介绍其中的一个：稠密连接网络（DenseNet） 。 它与ResNet的主要区别如图5.10所示。

![](/wiki/attach/images/d2lzh_notes/figure_5.10.png)

图5.10中将部分前后相邻的运算抽象为模块A和模块B。与ResNet的主要区别在于，DenseNet里模块B的输出不是像ResNet那样和模块A的输出相加，而是**在通道维上连结**。这样模块A的输出可以直接传入模块B后面的层。在这个设计里，**模块A直接跟模块B后面的所有层连接在了一起**。这也是它被称为“稠密连接”的原因。

DenseNet的主要构建模块是稠密块（dense block）和过渡层（transition layer）。前者定义了输入和输出是如何连结的，后者则用来控制通道数，使之不过大。

稠密块。DenseNet使用了ResNet改良版的“批量归一化、激活和卷积”结构（参见上一节的练习），我们首先在conv_block函数里实现这个结构：

```python
import d2lzh as d2l
from mxnet import gluon, init, nd
from mxnet.gluon import nn

def conv_block(num_channels):
    blk = nn.Sequential()
    blk.add(nn.BatchNorm(), nn.Activation('relu'),
            nn.Conv2D(num_channels, kernel_size=3, padding=1))
    return blk

# 稠密块由多个conv_block组成，每块使用相同的输出通道数。但在前向计算时，我们将每块的输入和输出在通道维上连结。
class DenseBlock(nn.Block):
    def __init__(self, num_convs, num_channels, **kwargs):
        super(DenseBlock, self).__init__(**kwargs)
        self.net = nn.Sequential()
        for _ in range(num_convs):
            self.net.add(conv_block(num_channels))

    def forward(self, X):
        for blk in self.net:
            Y = blk(X)
            X = nd.concat(X, Y, dim=1)  # 在通道维上将输入和输出连结
        return X

# 在下面的例子中，我们定义一个有2个输出通道数为10的卷积块。使用通道数为3的输入时，我们会得到通道数为3+2×10=233+2×10=23的输出。卷积块的通道数控制了输出通道数相对于输入通道数的增长，因此也被称为增长率（growth rate）。
blk = DenseBlock(2, 10)
blk.initialize()
X = nd.random.uniform(shape=(4, 3, 8, 8))
Y = blk(X)
Y.shape
# (4, 23, 8, 8)
```

过渡层。由于每个稠密块都会带来通道数的增加，使用过多则会带来过于复杂的模型。过渡层用来控制模型复杂度。它通过1×1卷积层来减小通道数，并使用步幅为2的平均池化层减半高和宽，从而进一步降低模型复杂度。

```python
def transition_block(num_channels):
    blk = nn.Sequential()
    blk.add(nn.BatchNorm(), nn.Activation('relu'),
            nn.Conv2D(num_channels, kernel_size=1),
            nn.AvgPool2D(pool_size=2, strides=2))
    return blk
```

DenseNet首先使用同ResNet一样的单卷积层和最大池化层。

类似于ResNet接下来使用的4个残差块，DenseNet使用的是4个稠密块。同ResNet一样，我们可以设置每个稠密块使用多少个卷积层。这里我们设成4，从而与上一节的ResNet-18保持一致。稠密块里的卷积层通道数（即增长率）设为32，所以每个稠密块将增加128个通道。

ResNet里通过步幅为2的残差块在每个模块之间减小高和宽。这里我们则使用过渡层来减半高和宽，并减半通道数。

同ResNet一样，最后接上全局池化层和全连接层来输出。
