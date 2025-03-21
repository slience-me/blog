---
layout: post
title: 论文笔记｜【论文笔记合集】TimesNet之FFT详解
categories: [论文笔记]
description: 【论文笔记合集】TimesNet之FFT详解
keywords: 机器学习, 论文笔记
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

![img](/images/posts/logo_slienceme3.png)

本文作者： [slience_me](https://slienceme.cn/)

---

# TimesNet之FFT详解

![image-20240119162800528](https://raw.githubusercontent.com/slience-me/picGo/master/images/image-20240119162800528.png)

## 1. 源代码

```python
def FFT_for_Period(x, k=2):
    xf = torch.fft.rfft(x, dim=1)
    frequency_list = abs(xf).mean(0).mean(-1)
    frequency_list[0] = 0
    _, top_list = torch.topk(frequency_list, k)
    top_list = top_list.detach().cpu().numpy()
    period = x.shape[1] // top_list
    return period, abs(xf).mean(-1)[:, top_list]
```

### 1.1 torch.fft.rfft(x, dim=1)

- **torch.fft.rfft()**

`torch.fft.rfft()` 是 PyTorch 中用于执行实数输入的快速傅里叶变换（FFT）的函数。该函数主要用于将实数输入转换为复数频谱。下面是一个详细解释和示例：

```python
import torch

# 创建一个实数输入的张量
input_tensor = torch.tensor([1.0, 2.0, 3.0, 4.0])

# 使用 torch.fft.rfft() 进行实数输入的快速傅里叶变换
complex_spectrum = torch.fft.rfft(input_tensor)

# 打印变换后的复数频谱
print("Complex Spectrum:", complex_spectrum)

# 获取频谱的实部和虚部
real_part = complex_spectrum.real
imaginary_part = complex_spectrum.imag

# 打印实部和虚部
print("Real Part:", real_part)
print("Imaginary Part:", imaginary_part)
```

**解释：**

1. 首先，我们创建一个包含实数值的 PyTorch 张量 `input_tensor`。
2. 然后，我们使用 `torch.fft.rfft()` 函数对该实数输入进行快速傅里叶变换，得到一个复数频谱 `complex_spectrum`。
3. 最后，我们打印复数频谱，并将其实部和虚部分别打印出来。

在这个例子中，`torch.fft.rfft()` 的输出是一个包含复数的张量，其中实部包含输入信号的正半轴频谱，而虚部则包含负半轴频谱。这种变换通常用于分析信号在频域中的特性。

**输出为：**

> Complex Spectrum: tensor([10.+0.j, -2.+2.j, -2.+0.j])
> Real Part: tensor([10., -2., -2.])
> Imaginary Part: tensor([0., 2., 0.])

**为什么四个输入得到3个输出？**

> 在傅里叶变换中，由于对称性，实数序列的傅里叶变换结果是具有一定规律的。对于长度为 `n` 的实数序列，其傅里叶变换结果的长度是 `n/2 + 1`。这是因为复数的共轭对称性，导致一半的频谱是冗余的。
>
> 在你的例子中，输入张量的长度为4，因此傅里叶变换的输出长度是 `4/2 + 1 = 3`。
>
> 如果你对具体的计算感兴趣，可以考虑查看傅里叶变换的相关数学理论。总体来说，这种长度减半的规律是由于信号是实数序列，而复数频谱是共轭对称的性质造成的。

**复数的共轭对称性**

> 复数的共轭对称性是指对于一个复数，它的共轭（conjugate）在复平面上关于实轴对称。对于复数 `a + bi`，其中 `a` 和 `b` 分别是实部和虚部，其共轭记为 `a - bi`。
>
> 更具体地说，如果有复数 `z = a + bi`，那么其共轭 `z*` 是 `a - bi`。这可以用下面的图示来表示：
>
> ```text
>  b |   .  * (a, b)
>      |   
>      |   
>  0 |------------------ a (实轴)
>      |   
>      | 
> -b|   .  * (a, -b)  <- 共轭对称
> ```
>
> 在频谱分析的上下文中，共轭对称性体现在傅里叶变换的频谱中。如果一个信号是实数序列，在频谱中正频率和负频率部分是共轭对称的。具体来说，如果某个频率分量 `f` 存在于正频率部分，那么 `-f` 就存在于负频率部分，且它们的振幅和相位是共轭关系。
>
> 这种对称性使得在频谱中只需存储一半的信息，因为另一半可以通过共轭关系获得。这也是为什么在实数序列的傅里叶变换中，输出的长度是输入长度的一半加一。



---

- **torch.fft.rfft(x, dim=1)**

`torch.fft.rfft(x, dim=1)` 是 PyTorch 中进行实数输入的一维快速傅里叶变换（FFT）的函数，其中 `dim=1` 指定了沿着哪个维度进行变换。

首先，假设 `x` 是一个张量，其中包含实数序列。通常情况下，`x` 的最后一个维度（`dim=-1`）应该是实数序列的维度。例如， `x` 的形状是 `(batch_size, 序列长度, 通道数)`，那么 `dim=-1` 就是在序列长度的维度上进行傅里叶变换。

现在，如果我们调用 `torch.fft.rfft(x, dim=1)`，它会在指定的维度上执行傅里叶变换。这就意味着对于输入张量 `x` 中的每个切片（沿着 `dim=1` 的方向），都会进行一维实数输入的快速傅里叶变换。

**示例：**

```python
import torch

# 假设 x 的形状为 (batch_size, sequence_length)
x = torch.randn(3, 4, 2)

# 在序列长度的维度上进行实数输入的一维 FFT
result = torch.fft.rfft(x, dim=1)

# 打印结果
# (3, 4//2+1, 2)
# (3, 3, 2)
print(result)
print(result.shape)
```

在这个示例中，`x` 是一个形状为 `(3, 4)` 的张量，其中 `dim=1` 表示在每个序列的长度维度上执行傅里叶变换。`result` 是变换后的结果。

需要注意的是，`torch.fft.rfft` 返回的结果是复数频谱的张量。如果需要获取实部和虚部，可以使用 `.real` 和 `.imag` 属性。

**输出结果为：**

> tensor([[[-1.4285+0.0000j,  0.6463+0.0000j],
>                [-1.8562+1.6229j,  1.0616+0.0355j],
>                [-3.8019+0.0000j,  1.3853+0.0000j]],
>
> ​              [[-2.6533+0.0000j, -2.2790+0.0000j],
> ​               [ 0.6657-0.6588j, -1.7569-1.6168j],
> ​               [-0.3022+0.0000j, -1.0560+0.0000j]],
>
> ​              [[-3.9908+0.0000j,  1.0519+0.0000j],
> ​               [-2.7840+0.2070j, -3.1494+0.0602j],
> ​               [-0.4287+0.0000j, -0.7184+0.0000j]]])
>
> torch.Size([3, 3, 2])



## 1.2 frequency_list = abs(xf).mean(0).mean(-1)

这一行代码涉及到对傅里叶变换结果的处理，计算频谱的平均振幅。让我们逐步解释：

1. `frequency_list = abs(xf).mean(0).mean(-1)`: 这一部分是对频谱进行处理：
   - `abs(xf)`: 计算复数频谱的振幅，即取绝对值。
   - `.mean(0)`: 沿着第一个维度（即 batch_size 维度）计算平均值，得到每个样本的平均振幅。
   - `.mean(-1)`: 沿着最后一个维度（即频率/通道/特征维度）计算平均值，得到每个样本、每个频率的平均振幅。

最终，`frequency_list` 包含了每个频率的平均振幅。这样的处理可以用于分析信号在频域上的能量分布，因为平均振幅反映了信号中各个频率成分的相对贡献。

举例来说，如果 `frequency_list` 的第 i 个元素表示第 i 个频率的平均振幅，那么可以通过分析这个列表来了解信号中哪些频率成分具有较大的能量。

**这样的好处是什么，为什么需要求frequency_list ？**

> 1. **频域分析：** 平均振幅提供了信号在频域上的分布信息。通过分析频率谱，你可以了解信号中各个频率成分的相对贡献。这对于理解信号的频率特征、检测特定频率的成分以及识别周期性模式都很有用。
>
> 2. **能量分布：** 平均振幅可以视为信号在不同频率上的能量分布。这对于识别信号中的主要频率、找到频域上的能量集中点以及确定信号的频率特征非常有帮助。
>
> 3. **特征提取：** 在某些应用中，频率分析可以用于提取信号的关键特征。通过计算频率谱的平均振幅，可以捕获信号中与频率相关的信息，这在一些信号处理任务中是有用的。
>
> 4. **滤波和去噪：** 通过频率分析，可以了解信号中的频率成分，从而进行滤波或去除噪音。对频率谱进行分析可以帮助确定哪些频率应该保留或去除，从而对信号进行处理。
>
> 总体而言，对频率谱进行分析和计算平均振幅可以提供对信号在频域上的理解，这对于很多信号处理和分析任务都是至关重要的。

## 1.3 frequency_list[0] = 0

将 `frequency_list[0]` 置零可能是为了去除直流分量（零频率分量）的影响。在频率谱中，索引为0的位置通常对应于零频率，也就是直流成分。

直流成分表示信号的平均值或偏移。在某些情况下，我们可能对信号的变化更感兴趣，而不是整个信号的平均值。通过将 `frequency_list[0]` 置零，我们可以去除直流成分的影响，更关注信号中的变化和其他频率成分。

这种操作在信号处理和频谱分析中是常见的，特别是当我们关注信号的变化或周期性成分而不关心平均水平时。它有助于突出频谱中的其他特征，使得分析更加集中于信号的变动和周期性。

**关于直流分量的解释**

> 直流分量是信号中的恒定成分，通常表示信号的平均值或直流偏移。直流分量不随时间变化，它是信号在水平方向上的偏移或平移。
>
> 让我们通过一个直观的例子来理解直流分量：
>
> 假设有一个以时间为横轴的信号图，其中纵轴表示信号的振幅。如果信号在整个时间范围内都有一个常数振幅，那么这个信号就包含直流分量。直流分量的存在会使整个信号在纵轴上发生平移，即整个信号的基准水平线上下移动。
>
> 举个例子，考虑一个表示温度的信号。如果这个信号中存在直流分量，那么它可能表示一个常数的环境温度，而信号的波动则表示温度随时间的变化。直流分量可以看作是整个信号的平均温度，而波动则反映了温度相对于平均值的变化。
>
> 在频谱分析中，直流分量通常对应于频谱中的零频率分量，即索引为0的位置。通过将直流分量从频谱中去除，我们可以更专注于信号中变化的频率成分，而不受整体平移的影响。这在很多信号处理任务中是有用的，特别是当我们关注信号的变动和周期性成分时。

## 1.4 _, top_list = torch.topk(frequency_list, k)

`torch.topk` 是 PyTorch 中用于获取张量中最大的 k 个元素的函数。frequency_list` 是一个包含频率振幅的张量，而 `_` 和 `top_list` 是函数的返回结果。

让我们逐步解释这行代码：

```python
_, top_list = torch.topk(frequency_list, k)
```

- `torch.topk`: 这个函数用于获取张量中最大的 k 个元素。它返回两个张量，第一个是最大值的值（在这里我们用 `_` 表示忽略，因为我们不使用这个值），第二个是最大值对应的索引。

- `frequency_list`: 这是包含频率振幅的张量，其中每个元素表示某个频率的平均振幅。

- `k`: 这是要获取的最大元素的数量。

- `_, top_list`: 这是用来接收 `torch.topk` 函数的返回结果。`_` 用于忽略最大值的值，而 `top_list` 包含最大值对应的索引。

总体而言，这行代码的目的是从 `frequency_list` 中找到最大的 k 个频率振幅，并获取这些最大值对应的索引。这可以用于找到信号中主要的频率成分。如果 `top_list` 包含的是频率的索引，你可以通过这些索引查找对应的频率值。

**举例：**

假设我们有一个包含频率振幅的张量 `frequency_list`，内容如下：

```python
import torch

frequency_list = torch.tensor([5.0, 2.0, 8.0, 3.0, 1.0])
```

现在，我们想找到最大的两个频率振幅对应的索引。我们可以使用 `torch.topk` 函数来实现：

```python
k = 2
_, top_list = torch.topk(frequency_list, k)

print("Top k frequencies indices:", top_list)
```

输出是：

```
Top k frequencies indices: tensor([2, 0])
```

这表示在 `frequency_list` 中，最大的两个频率振幅对应的索引分别是2和0。也就是说，频率振幅最大的是索引为2的元素（8.0），其次是索引为0的元素（5.0）。

## 1.5 top_list = top_list.detach().cpu().numpy()

这行代码对 `top_list` 进行了一系列操作，将其从 PyTorch 的张量类型转换为 NumPy 数组。让我们逐步解释这些操作：

```python
top_list = top_list.detach().cpu().numpy()
```

- `top_list.detach()`: `detach()` 方法用于创建一个没有梯度信息的张量副本。在 PyTorch 中，张量的梯度信息通常用于自动微分。`detach()` 可以用于生成新的张量，该张量与原始张量共享数据，但没有梯度信息。

- `.cpu()`: 如果张量存储在 GPU 上，`cpu()` 方法用于将其移到 CPU 上。在这里，这一步可能是为了确保张量在 CPU 上，以便进行 NumPy 转换。

- `.numpy()`: `numpy()` 方法用于将 PyTorch 张量转换为 NumPy 数组。这是因为 PyTorch 和 NumPy 是两个不同的库，有时需要在它们之间进行数据转换。

综合这些步骤，`top_list` 最终被转换为一个不带梯度信息的 CPU 上的 NumPy 数组。这样的转换通常是为了在 PyTorch 和 NumPy 之间进行数据交互，因为它们在许多方面具有互操作性。 NumPy 数组是 Python 中广泛使用的数据结构，可以用于进行各种科学计算和数据分析任务。

## 1.6 period = x.shape[1] // top_list

这行代码计算了一个名为 `period` 的值，它是 `x` 张量的第二个维度（即序列长度）除以 `top_list` 中每个元素的值。这通常是用于计算信号中特定频率成分的周期。

让我们分解这行代码：

```python
period = x.shape[1] // top_list
```

- `x.shape[1]`: 这是张量 `x` 的第二个维度的长度，即序列长度。在这个上下文中，我们假设 `x` 的形状是 `(batch_size, sequence_length)`。

- `top_list`: 这是包含最大频率振幅的索引的列表。每个索引对应于在 `x` 中找到的重要频率。

- `//`: 这是整数除法运算符，返回除法的整数部分。在这里，它用于计算 `x.shape[1]` 除以 `top_list` 中的每个元素。

最终，`period` 将包含每个最大频率对应的周期。例如，如果 `top_list` 中的某个元素是2，那么 `period` 中对应的值将是 `x` 中的信号在该频率上的周期长度。这样的计算可以用于分析信号的周期性成分。

## 1.7 return period, abs(xf).mean(-1)[:, top_list]

这行代码包含了一个返回语句，返回两个值：`period` 和一个部分截取的频谱信息。让我们逐步解释这行代码：

```python
return period, abs(xf).mean(-1)[:, top_list]
```

1. `period`: 这是之前计算的周期，表示信号中每个最大频率成分的周期长度。

2. `abs(xf).mean(-1)[:, top_list]`：
   - `abs(xf)`: 先取复数频谱的振幅，即绝对值。这表示我们对频谱的振幅部分感兴趣。
   - `.mean(-1)`: 沿着最后一个维度（通常是频率维度）计算平均值，得到每个样本、每个频率的平均振幅。这部分可以看作是对整个频谱的平均振幅信息。
   - `[:, top_list]`: 通过索引 `top_list`，选择仅包含最大频率成分的部分。

综合起来，这行代码返回了信号中每个最大频率成分的周期长度以及相应频率成分的平均振幅信息。这样的返回结果可能用于进一步分析信号中不同频率的周期性成分及其振幅特征。
