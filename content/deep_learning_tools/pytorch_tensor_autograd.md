---
title: "Pytorch Tensor and Autograd"
date: 2019-07-09 01:01
---

## Tensor

在PyTorch中，`torch.Tensor` 是存储和变换数据的主要工具。 Tensor 和NumPy的多维数组非常类似。然而，Tensor 提供 GPU 计算和自动求梯度等更多功能，使其更加适合深度学习。 

```python
import torch

x = torch.empty(3, 4)
print(x)
# tensor([[4.1497e-08, 2.6804e-09, 1.2541e+16, 1.2849e+31],
#         [1.8395e+25, 6.1963e-04, 3.2907e+21, 8.1934e-10],
#         [2.1744e+23, 2.6730e-06, 1.7079e-07, 4.2468e-08]])

# 随机初始化
x = torch.rand(5, 3)

# 创建一个 5x3 的 long 型全 0 的 Tensor
x = torch.zeros(5, 3, dtype=torch.long)

# 直接根据数据创建
x = torch.tensor([5.5, 3])

# 通过现有的Tensor来创建，此方法会默认重用输入Tensor的一些属性，例如数据类型，除非自定义数据类型。
x = x.new_ones(5, 3, dtype=torch.float64)  # 返回的tensor默认具有相同的torch.dtype和torch.device
x = torch.randn_like(x, dtype=torch.float) # 指定新的数据类型

# 可以通过shape或者size()来获取Tensor的形状:
print(x.size())
print(x.shape)
```

Pytorch 算术操作

```python
x = torch.rand(5, 3)
y = torch.rand(5, 3)
print(x + y)

print(torch.add(x, y))

result = torch.empty(5, 3)
torch.add(x, y, out=result)
print(result)

# inplace 形式
y.add_(x)
print(y)

# Pytorch 的大部分操作都用 inplace 形式，用后缀 `_` 标识，如
x.copy_(y)  # 将 y 的值拷贝到 x 中
x.t_()  # 将 x 转置并赋值
```

Pytorch tensor 的索引

可以使用类似NumPy的索引操作来访问Tensor的一部分，需要注意的是： **索引出来的结果与原数据共享内存，也即修改一个，另一个会跟着修改** 。

```python
y = x[0, :]
y += 1
print(y)
print(x)
```

用 `view()` 来改变 `Tensor` 的形状：

```python
x = torch.ones(3, 4)
print(x)

y = x.view(3 * 4)
print(y)

y = x.view(-1, 2)
print(y)
```

注意view()返回的新Tensor与源Tensor虽然可能有不同的size，但是是共享data的，也即更改其中的一个，另外一个也会跟着改变。(顾名思义，view仅仅是改变了对这个张量的观察角度，内部数据并未改变)

所以如果我们想返回一个真正新的副本（即不共享data内存）该怎么办呢？Pytorch还提供了一个reshape()可以改变形状，但是此函数并不能保证返回的是其拷贝，所以不推荐使用。推荐先用clone创造一个副本然后再使用view。

```python
x_cp = x.clone().view(15)
x -= 1
print(x)
print(x_cp)
```

使用clone还有一个好处是会被记录在计算图中，即梯度回传到副本时也会传到源Tensor。

另外一个常用的函数就是item(), 它可以将一个标量Tensor转换成一个Python number：

```python
x = torch.randn(1)
print(x)
print(x.item())
```

广播机制

当对两个形状不同的Tensor按元素运算时，可能会触发广播（broadcasting）机制：先适当复制元素使这两个Tensor形状相同后再按元素运算。


Tensor 和 NumPy 相互转换

我们很容易用 `numpy()` 和 `from_numpy()` 将Tensor和NumPy中的数组相互转换。但是需要注意的一点是： 这两个函数所产生的的Tensor和NumPy中的数组共享相同的内存（所以他们之间的转换很快），改变其中一个时另一个也会改变！！！

数据转到 GPU 上，用 `to`

```python
# 以下代码只有在PyTorch GPU版本上才会执行
if torch.cuda.is_available():
    device = torch.device("cuda")          # GPU
    y = torch.ones_like(x, device=device)  # 直接创建一个在GPU上的Tensor
    x = x.to(device)                       # 等价于 .to("cuda")
    z = x + y
    print(z)
    print(z.to("cpu", torch.double))       # to()还可以同时更改数据类型
```

---

## Autograd

Pytorch 中的 autograd。

Pytorch 中的 Tensor 支持自动求导。

如果 Tensor 的属性 `.requires_grad` 设置为 True ，它将开始追踪(track)在其上的所有操作（这样就可以利用链式法则进行梯度传播了）。

完成计算后，可以调用 `.backward()` 来完成所有梯度计算。此 Tensor 的梯度将累积到 `.grad` 属性中。

注意在 `y.backward()` 时，如果 `y` 是标量，则不需要为 `backward()` 传入任何参数；否则，需要传入一个与 `y` 同形的 Tensor。

如果不想要被继续追踪，可以调用 `.detach()` 将其从追踪记录中分离出来，这样就可以防止将来的计算被追踪，这样梯度就传不过去了。此外，还可以用 `with torch.no_grad()` 将不想被追踪的操作代码块包裹起来，这种方法在评估模型的时候很常用，因为在评估模型时，我们并不需要计算可训练参数（ `requires_grad=True` ）的梯度。

`Function` 是另外一个很重要的类。Tensor和Function互相结合就可以构建一个记录有整个计算过程的有向无环图（DAG）。每个Tensor都有一个 `.grad_fn` 属性，该属性即创建该Tensor的Function, 就是说该Tensor是不是通过某些运算得到的，若是，则grad_fn返回一个与这些运算相关的对象，否则是None。
