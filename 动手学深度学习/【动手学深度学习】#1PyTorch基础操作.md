> 主要参考学习资料：
> 
> 《动手学深度学习》阿斯顿·张 等 著
> 
> 【动手学深度学习 PyTorch版】哔哩哔哩@跟李牧学AI

@[TOC](目录)
# 1.1数据操作

## 1.1.1入门

arange方法创建行向量x：


```python
import torch
x = torch.arange(12)
x
```




    tensor([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11])



shape属性访问张量的形状（沿每个轴的长度）：


```python
x.shape
```




    torch.Size([12])



numel属性访问张量的元素总数：


```python
x.numel()
```




    12



reshape方法改变张量的形状但不改变大小：


```python
X = x.reshape(3, 4)
X
```




    tensor([[ 0,  1,  2,  3],
            [ 4,  5,  6,  7],
            [ 8,  9, 10, 11]])



zeros方法创建元素均为0的张量：


```python
torch.zeros((2, 3, 4))
```




    tensor([[[0., 0., 0., 0.],
             [0., 0., 0., 0.],
             [0., 0., 0., 0.]],
    
            [[0., 0., 0., 0.],
             [0., 0., 0., 0.],
             [0., 0., 0., 0.]]])



ones方法创建元素均为1的张量：


```python
torch.ones((2, 3, 4))
```




    tensor([[[1., 1., 1., 1.],
             [1., 1., 1., 1.],
             [1., 1., 1., 1.]],
    
            [[1., 1., 1., 1.],
             [1., 1., 1., 1.],
             [1., 1., 1., 1.]]])



randn方法创建元素从标准高斯分布采样的张量：


```python
torch.randn(3, 4)
```




    tensor([[ 0.0058, -0.7264, -1.1605, -1.5227],
            [-0.9009,  0.8884, -0.2270, -0.4142],
            [ 1.1712, -0.1724, -0.5896,  0.8420]])


tensor方法使用Python列表创建元素被赋予确定值的张量：

```python
torch.tensor([[2, 1, 4, 3], [1, 2, 3, 4], [4, 3, 2, 1]])
```




    tensor([[2, 1, 4, 3],
            [1, 2, 3, 4],
            [4, 3, 2, 1]])



## 1.1.2运算符

常见的标准算术运算符对于具有相同形状的张量可以被升级为按元素运算：


```python
x = torch.tensor([1.0, 2, 4, 8])
y = torch.tensor([2, 2, 2, 2])
x + y, x - y, x * y, x / y, x ** y
```




    (tensor([ 3.,  4.,  6., 10.]),
     tensor([-1.,  0.,  2.,  6.]),
     tensor([ 2.,  4.,  8., 16.]),
     tensor([0.5000, 1.0000, 2.0000, 4.0000]),
     tensor([ 1.,  4., 16., 64.]))



cat方法将多个张量连接在一起，dim参数指定按哪个轴连接：


```python
X = torch.arange(12, dtype=torch.float32).reshape((3, 4))
Y = torch.tensor([[2.0, 1, 4, 3], [1, 2, 3, 4], [4, 3, 2, 1]])
torch.cat((X, Y), dim=0), torch.cat((X, Y), dim=1)
```




    (tensor([[ 0.,  1.,  2.,  3.],
             [ 4.,  5.,  6.,  7.],
             [ 8.,  9., 10., 11.],
             [ 2.,  1.,  4.,  3.],
             [ 1.,  2.,  3.,  4.],
             [ 4.,  3.,  2.,  1.]]),
     tensor([[ 0.,  1.,  2.,  3.,  2.,  1.,  4.,  3.],
             [ 4.,  5.,  6.,  7.,  1.,  2.,  3.,  4.],
             [ 8.,  9., 10., 11.,  4.,  3.,  2.,  1.]]))



逻辑运算符按元素判断并构建布尔张量：


```python
X == Y
```




    tensor([[False,  True, False,  True],
            [False, False, False, False],
            [False, False, False, False]])



sum方法对张量中的所有元素求和：


```python
X.sum()
```




    tensor(66.)



## 1.1.3广播机制

形状不同的张量通过调用广播机制来执行按元素操作，其工作方式如下：

1. 通过适当复制元素来扩展一个或两个数组，以便在转换之后两个张量具有相同的形状；

2. 对生成的数组执行按元素操作。


```python
a = torch.arange(3).reshape((3, 1))
b = torch.arange(2).reshape((1, 2))
a, b
```




    (tensor([[0],
             [1],
             [2]]),
     tensor([[0, 1]]))




```python
a + b
```




    tensor([[0, 1],
            [1, 2],
            [2, 3]])



## 1.1.4索引和切片

张量中的元素可以通过索引访问。第一个元素的索引是0，最后一个元素的索引是-1，可以通过左闭右开的切片指定范围：


```python
X[-1], X[1:3]
```




    (tensor([ 8.,  9., 10., 11.]),
     tensor([[ 4.,  5.,  6.,  7.],
             [ 8.,  9., 10., 11.]]))



通过指定索引对元素赋值：


```python
X[1, 2] = 9
X
```




    tensor([[ 0.,  1.,  2.,  3.],
            [ 4.,  5.,  9.,  7.],
            [ 8.,  9., 10., 11.]])



通过切片对元素批量赋值：


```python
X[0:2, :] = 12
X
```




    tensor([[12., 12., 12., 12.],
            [12., 12., 12., 12.],
            [ 8.,  9., 10., 11.]])



## 1.1.5节省内存

使用[:]不会为结果分配新的内存，进而减少操作的内存开销：


```python
before = id(X)
X[:] = X + Y
id(X) == before
```




    True



## 1.1.6转换为其他Python对象

torch张量和numpy数组可以相互转换：


```python
A = X.numpy()
B = torch.tensor(A)
type(A), type(B)
```




    (numpy.ndarray, torch.Tensor)



使用item方法或Python的内置函数将大小为1的张量转换为Python标量：


```python
a = torch.tensor([3.5])
a, a.item(), float(a), int(a)
```




    (tensor([3.5000]), 3.5, 3.5, 3)



# 1.2数据预处理

## 1.2.1读取数据集

创建一个人工数据集并存储在CSV（逗号分隔值）文件中，每一列按逗号分隔，第一行为表头，NA为缺失数据：


```python
import os

os.makedirs(os.path.join('..', 'data'), exist_ok=True)
data_file = os.path.join('..', 'data', 'house_tiny.csv')
with open(data_file, 'w') as f:
    f.write('NumRooms,Alley,Price\n')
    f.write('NA,Pave,127500\n')
    f.write('2,NA,106000\n')
    f.write('4,NA,178100\n')
    f.write('NA,NA,140000\n')
```

pandas库的read_csv方法加载原始数据集：


```python
import pandas as pd

data = pd.read_csv(data_file)
print(data)
```

       NumRooms Alley   Price
    0       NaN  Pave  127500
    1       2.0   NaN  106000
    2       4.0   NaN  178100
    3       NaN   NaN  140000
    

## 1.2.2处理缺失值

通过位置索引iloc将data的前两列存入inputs，最后一列存入outputs，并用同一列的均值替换inputs中的数值缺失项：


```python
inputs, outputs = data.iloc[:, 0:2], data.iloc[:, 2]
inputs = inputs.fillna(inputs.mean(numeric_only=True))
print(inputs)
```

       NumRooms Alley
    0       3.0  Pave
    1       2.0   NaN
    2       4.0   NaN
    3       3.0   NaN
    

根据字符串的值将Alley分为Pave和NaN两类，并用数值代替布尔值标记特征：


```python
inputs = pd.get_dummies(inputs, dummy_na=True, dtype=int)
print(inputs)
```

       NumRooms  Alley_Pave  Alley_nan
    0       3.0           1          0
    1       2.0           0          1
    2       4.0           0          1
    3       3.0           0          1
    

## 1.2.3转换为张量格式

使用tensor方法将条目转换为张量：


```python
X, y = torch.tensor(inputs.values), torch.tensor(outputs.values)
X, y
```




    (tensor([[3., 1., 0.],
             [2., 0., 1.],
             [4., 0., 1.],
             [3., 0., 1.]], dtype=torch.float64),
     tensor([127500, 106000, 178100, 140000]))



# 1.3线性代数

## 1.3.1标量

用只有一个元素的张量表示标量：


```python
x = torch.tensor(3.0)
y = torch.tensor(2.0)

x + y, x * y, x / y, x ** y
```




    (tensor(5.), tensor(6.), tensor(1.5000), tensor(9.))



## 1.3.2向量

用只有一个轴的张量表示向量：


```python
x = torch.arange(4)
x
```




    tensor([0, 1, 2, 3])



使用下标访问向量的任一元素：


```python
x[3]
```




    tensor(3)



使用len函数访问向量的长度：


```python
len(x)
```




    4



使用shape属性访问向量的长度：


```python
x.shape
```




    torch.Size([4])



## 1.3.3矩阵

使用reshape方法指定两个分量$m$和$n$来创建一个形状为$m\times n$的矩阵：


```python
A = torch.arange(20).reshape(5, 4)
A
```




    tensor([[ 0,  1,  2,  3],
            [ 4,  5,  6,  7],
            [ 8,  9, 10, 11],
            [12, 13, 14, 15],
            [16, 17, 18, 19]])



使用T方法访问矩阵的转置：


```python
A.T
```




    tensor([[ 0,  4,  8, 12, 16],
            [ 1,  5,  9, 13, 17],
            [ 2,  6, 10, 14, 18],
            [ 3,  7, 11, 15, 19]])



## 1.3.4张量

通过张量实现更多轴的数据结构：


```python
X = torch.arange(24).reshape(2, 3, 4)
X
```




    tensor([[[ 0,  1,  2,  3],
             [ 4,  5,  6,  7],
             [ 8,  9, 10, 11]],
    
            [[12, 13, 14, 15],
             [16, 17, 18, 19],
             [20, 21, 22, 23]]])



## 1.3.5张量算法的基本性质

张量的元素加法和元素乘法（Hadamard乘积）不改变张量的形状：


```python
A = torch.arange(20, dtype=torch.float32).reshape(5, 4)
B = A.clone()
A, A + B
```




    (tensor([[ 0.,  1.,  2.,  3.],
             [ 4.,  5.,  6.,  7.],
             [ 8.,  9., 10., 11.],
             [12., 13., 14., 15.],
             [16., 17., 18., 19.]]),
     tensor([[ 0.,  2.,  4.,  6.],
             [ 8., 10., 12., 14.],
             [16., 18., 20., 22.],
             [24., 26., 28., 30.],
             [32., 34., 36., 38.]]))




```python
A * B
```




    tensor([[  0.,   1.,   4.,   9.],
            [ 16.,  25.,  36.,  49.],
            [ 64.,  81., 100., 121.],
            [144., 169., 196., 225.],
            [256., 289., 324., 361.]])



张量与标量相加或相乘时，张量的每个元素和标量相加或相乘，不改变形状：


```python
a = 2
X = torch.arange(24).reshape(2, 3, 4)
a + X, (a * X).shape
```




    (tensor([[[ 2,  3,  4,  5],
              [ 6,  7,  8,  9],
              [10, 11, 12, 13]],
     
             [[14, 15, 16, 17],
              [18, 19, 20, 21],
              [22, 23, 24, 25]]]),
     torch.Size([2, 3, 4]))



## 1.3.6降维

sum方法计算任意形状张量的元素和使之变为一个标量，相当于降维：


```python
x = torch.arange(4, dtype=torch.float32)
x, x.sum()
```




    (tensor([0., 1., 2., 3.]), tensor(6.))




```python
A.shape, A.sum()
```




    (torch.Size([5, 4]), tensor(190.))



axis参数指定张量沿哪一个轴通过求和降低维度：


```python
A_sum_axis0 = A.sum(axis=0)
A_sum_axis0, A_sum_axis0.shape
```




    (tensor([40., 45., 50., 55.]), torch.Size([4]))




```python
A_sum_axis1 = A.sum(axis=1)
A_sum_axis1, A_sum_axis1.shape
```




    (tensor([ 6., 22., 38., 54., 70.]), torch.Size([5]))



沿着行和列对矩阵求和等价于对矩阵的所有元素求和：


```python
A.sum(axis=[0, 1])
```




    tensor(190.)



mean方法将总和除以元素总数来计算张量的平均值：


```python
A.mean(), A.sum() / A.numel()
```




    (tensor(9.5000), tensor(9.5000))



mean方法也可以沿指定轴降低张量的维度：


```python
A.mean(axis=0), A.sum(axis=0) / A.shape[0]
```




    (tensor([ 8.,  9., 10., 11.]), tensor([ 8.,  9., 10., 11.]))



将keepdims参数设为True在求和或平均值时保持轴数不变有时会很有用：


```python
sum_A = A.sum(axis=1, keepdims=True)
sum_A
```




    tensor([[ 6.],
            [22.],
            [38.],
            [54.],
            [70.]])



保持轴数不变可以通过广播求得每个元素的数值在各自行中的占比：


```python
A / sum_A
```




    tensor([[0.0000, 0.1667, 0.3333, 0.5000],
            [0.1818, 0.2273, 0.2727, 0.3182],
            [0.2105, 0.2368, 0.2632, 0.2895],
            [0.2222, 0.2407, 0.2593, 0.2778],
            [0.2286, 0.2429, 0.2571, 0.2714]])



cumsum方法沿某个轴逐一累积元素总和而不会降维：


```python
A.cumsum(axis=0)
```




    tensor([[ 0.,  1.,  2.,  3.],
            [ 4.,  6.,  8., 10.],
            [12., 15., 18., 21.],
            [24., 28., 32., 36.],
            [40., 45., 50., 55.]])



## 1.3.7点积

dot方法计算向量点积：


```python
y = torch.ones(4, dtype=torch.float32)
x, y, torch.dot(x, y)
```




    (tensor([0., 1., 2., 3.]), tensor([1., 1., 1., 1.]), tensor(6.))



也可以通过按元素乘法后求和来表示：


```python
torch.sum(x * y)
```




    tensor(6.)



## 1.3.8矩阵-向量积

mv方法计算矩阵和向量的乘法：


```python
A.shape, x.shape, torch.mv(A, x)
```




    (torch.Size([5, 4]), torch.Size([4]), tensor([ 14.,  38.,  62.,  86., 110.]))



## 1.3.9矩阵-矩阵乘法

mm方法计算矩阵乘法：


```python
B = torch.ones(4, 3)
torch.mm(A, B)
```




    tensor([[ 6.,  6.,  6.],
            [22., 22., 22.],
            [38., 38., 38.],
            [54., 54., 54.],
            [70., 70., 70.]])



## 1.3.10范数

norm方法计算向量的$L_2$范数：


```python
u = torch.tensor([3.0, -4.0])
torch.norm(u)
```




    tensor(5.)



绝对值和求和组合计算向量的$L_1$范数：


```python
torch.abs(u).sum()
```




    tensor(7.)



norm方法也能计算矩阵的Frobenius范数：


```python
torch.norm(torch.ones((4, 9)))
```




    tensor(6.)



# 1.4自动微分

## 1.4.1标量变量的反向传播

torch根据设计好的模型构建一个计算图来跟踪计算是哪些数据通过哪些操作组合起来产生输出，并通过自动微分反向传播梯度，即跟踪整个计算图，填充关于每个参数的偏导数。


```python
x = torch.arange(4.0)
x
```




    tensor([0., 1., 2., 3.])



将requires_grad_属性设为True来存储梯度，grad方法计算梯度（默认为None）：


```python
x.requires_grad_(True)
x.grad
```

令$\boldsymbol y=2\boldsymbol x^\top\boldsymbol x$：


```python
y = 2 * torch.dot(x, x)
y
```




    tensor(28., grad_fn=<MulBackward0>)



调用backward函数自动计算$\boldsymbol y$关于$\boldsymbol x$每个分量的梯度：


```python
y.backward()
x.grad
```




    tensor([ 0.,  4.,  8., 12.])



验证结果：


```python
x.grad == 4 * x
```




    tensor([True, True, True, True])



pytorch默认累积梯度，计算另一个函数的梯度之前调用zero_方法清楚之前的值：


```python
x.grad.zero_()
y = x.sum()
y.backward()
x.grad
```




    tensor([1., 1., 1., 1.])



## 1.4.2非标量变量的反向传播

对于高阶的y和x，求导的结果可以是一个高阶张量。但通常我们不会计算微分矩阵，而是通过结合sum方法单独计算每个样本的偏导数：


```python
x.grad.zero_()
y = x * x
y.sum().backward()
x.grad
```




    tensor([0., 2., 4., 6.])



## 1.4.3分离计算

detach方法实现分离计算，在计算z关于x的梯度时将y视为一个常数，忽视y作为x的函数的计算：


```python
x.grad.zero_()
y = x * x
u = y.detach()
z = u * x

z.sum().backward()
x.grad == u
```




    tensor([True, True, True, True])



## 1.4.4Python控制流的梯度计算

即使构建函数的计算图需要通过Python控制流，我们也可以计算得到变量的梯度。下列函数对a只是进行了线性分段的加倍操作，其对f(a)反向传播的梯度将是a最终的系数：


```python
def f(a):
    b = a * 2
    while b.norm() < 1000:
        b = b * 2
    if b.sum() > 0:
        c = b
    else:
        c = 100 * b
    return c
```


```python
a = torch.randn(size=(), requires_grad=True)
d = f(a)
d.backward()
```


```python
a.grad == d / a
```




    tensor(True)



