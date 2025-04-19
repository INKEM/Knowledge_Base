> 本文力图用最快的方式向大家陈列Python的基础语法，适合接触过其他编程语言后快速上手Python或供查阅巩固用
> 
> 参考书籍：《Python程序设计 人工智能案例实践》[美] 保罗·戴特尔 哈维·戴特尔 著
>
> 码字不易，求点赞收藏加关注
>
>有问题欢迎评论区讨论

@[TOC](目录)

# Python基础语法速览（下）

## 字典和集合

### 字典

**字典**是无序的**键-值对**的合集，像传统字典中字和释义的映射一样，键-值对存储不可变的键到值的映射。

字典的键必须是不可变且唯一的，多个键可以具有相同的值。

#### 创建字典

```python
In [1]: country_codes = {'Finland': 'fi', 'South Africa': 'za', 'Nepal': 'np'}
#用{}和逗号分隔的“键: 值”创建一个用国家名称作为键的字典，对应互联网国家代码值

In [2]: country_codes
Out[2]: {'Finland': 'fi', 'South Africa': 'za', 'Nepal': 'np'}

In [3]: len(country_codes) #内置函数len返回字典中键-值对的数量
Out[3]: 3

In [4]: if country_codes: #非空的字典等价于True，空字典等价于False
   ...: 	print('country_codes is not empty')
   ...: else:
   ...: 	print('country_codes is empty')
country_codes is not empty

In [5]: country_codes.clear() #清空字典
```

#### 遍历字典

```python
days_per_month = {'January': 31, 'February': 28, 'March': 31}
for month,days in days_per_month.items(): #items方法返回由键-值对构成的元组
	print(f'{month} has {days} days')
------
January has 31 days
February has 28 days
March has 31 days
```

#### 基本的字典操作

```python
In [1]: roman_numerals = {'I': 1, 'II': 2, 'III': 3, 'V': 5, 'X': 100}

In [2]: roman_numerals['V'] #获取键'V'对应的值
Out[2]: 5

In [3]: roman_numerals['X'] = 10 #更新与键'X'关联的值

In [4]: roman_numerals['L'] = 50 #给不存在的键复制会在字典中插入新的键-值对

In [5]: del roman_numerals['III'] #del语句从字典中删除键-值对

In [6]: roman_numerals.pop['X'] #pop方法删除键-值对的同时返回已删除键的值
Out[6]: 10

In [7]: roman_numerals.get('III') #get方法通常返回其参数的对应值，如果找不到该键则返回None，可避免KeyError错误

In [8]: roman_numerals.get('III', 'III not in dictionary') #第二个参数是未找到键时返回的自定义消息
Out[8]: III not in dictionary

In [9]: 'V' in roman_numerals #运算符in和not in用于确定字典是否包含指定的键
Out[9]: True

In [10]: 'III' not in roman_numerals
Out[10]: 'True'
```

#### 字典的keys和values方法

```python
In [1]: months = {'January': 1, 'February': 2, 'March': 3}

In [2]: for month_name in months.keys(): #keys方法返回字典的键
   ...: 	print(month_name, end = ' ')
January February March 

In [3]: for month_number in months.values(): #values方法返回字典的值
   ...: 	print(month_name, end = ' ')
1 2 3 
```

字典的items、keys、values方法返回的是字典的**数据视图**而不是单纯复制出一个数据副本，因此储存其返回值的变量能反映字典当前的实际内容：

```python
In [4]: months_view = months.keys()

In [5]: months['December'] = 12

In [6]: for key in months_view:
   ...: 	print(key, end = ' ')
January February March December
#即使没有对months_view重新赋值，其依旧储存了字典更新后的内容
```

```python
#以下内容将字典的键、值和键-值对转换为列表
In [7]: list(months.keys())
Out[7]: ['January', 'February', 'March', 'December']

In [7]: list(months.values())
Out[7]: [1, 2, 3, 12]

In [7]: list(months.items())
Out[7]: [('January', 1), ('February', 2), ('March', 3), ('December', 12)]

#内置sorted函数可以按字母顺序遍历键
In [8]: for month_name in sorted(month.keys())
   ...: 	print(month_name, end = ' ')
December February January March
```

#### 字典的比较

比较运算符==和!=分别用于确定两个字典是否具有相同或不同的内容，当两个字典具有相同的键-值对（无论顺序）时，相等运算符返回True。

```python
In [1]: country_capitals1 = {'Belgium': 'Brussels', 'Haiti': 'Port-au-Prince'}

In [2]: country_capitals2 = {'Nepal': 'Kathmandu', 'Uruguay': 'Montevideo'}

In [3]: country_capitals3 = {'Haiti': 'Port-au-Prince', 'Belgium': 'Brussels'}

In [4]: country_capitals1 == country_capitals3
Out[4]: True

In [5]: country_caputals1 != country_capitals2
Out[5]: False
```

#### 字典的update方法

```python
In [1]: country_codes = {}

In [2]: country_codes.update({'South Africa': 'za'}) #插入键-值对

In [3]: country_codes.update(Australia = 'ar') #自动将参数名称转换为字符串键并插入键-值对

In [4]: country_codes.update(Australia = 'au') #更新与'Austrailia'相关的值
```

#### 字典推导式

**字典推导式**为快速生成字典提供了一种方便的表示方法，通常是将一个字典映射到另一个字典。

```python
In [1]: months = {'January': 1, 'February': 2, 'March': 3}

In [2]: months2 = {number: name for name, number in months.items()}
# for左边的表达式指定了key: value形式的键-值对，右边遍历months.items()将每个键-值对解包到变量name和number中，得到一个从月份数映射到月份名的新字典

In [3]: grades = {'Sue': [98, 87, 94], 'Bob': [84, 95, 91]}

In [4]: grades2 = {k: sum(v) / len(v) for k, v in grades.items()}
#推导式将grades.items()返回的每个元组解包为k（名称）和v（成绩列表），并用键k和sum(v)/len(v)的值创建一个新的键-值对，得到一个从学生姓名映射到平均成绩的新字典
```

### 集合

**集合**是元素值不重复的无序合集，只可以包含不可变的元素，比如字符串、整型、浮点数和元组。

#### 创建集合

```python
In [1]: colors = {'red', 'orange', 'yellow', 'green', 'red', 'blue'} #创建一个颜色的字符串集合

In [2]: len(colors) #内置len函数确定一个集合中元素的数量
Out[2]: 5

In [3]: 'red' in colors #使用in和not in操作符检查一个集合是否包含特定的值
Out[3]: True

In [4]: 'purple' not in colors
Out[4]: True

In [5]: for color in colors: #集合可迭代，可用for循环处理每个元素
   ...: 	print(color.upper(), end = ' ')
RED GREEN YELLOW BLUE ORANGE #集合是无序的，迭代处理方法不能依赖元素访问顺序

In [6]: numbers = list(range(10)) + list(range(5))
#[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4]

In [7]: set(numbers) #内置set函数根据一组元素值创建一个集合，集合会合并重复的元素
Out[7]: {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

In [8]: set() #创建空集合必须用set()函数，空花括号代表空字典
Out[8]: set()
```

集合是可变的，即可以添加或删除元素，但集合元素不可变，所以一个集合不能将其他集合作为元素，但**frozenset**是一个不可变的集合，即创建之后不能被修改，所以一个集合可以包含frozenset对象作为元素。

#### 集合的比较

```python
#==和!=运算符检验集合是否包含相同的值
In [1]: {1, 3, 5} == {3, 5, 1}
Out[1]: True

In [2]: {1, 3, 5} != {3, 5, 1}
Out[2]: False

#<运算符检验左边的集合是否是右边集合的真子集
In [3]: {1, 3, 5} < {3, 5, 1}
Out[3]: False

In [4]: {1, 3, 5} < {7, 3, 5, 1}
Out[4]: True

#<=运算符检验左边的集合是否是右边集合的非真子集
In [5]: {1, 3, 5} <= {3, 5, 1}
Out[5]: True

In [6]: {1, 3, 5} <= {7, 3, 5, 1}
Out[6]: True

#也可用集合的issubset方法检验非真子集
In [7]: {1, 3, 5}.issubset({3, 5, 1})
Out[7]: True

#>和>=运算符分别检验左边的集合是否是右边集合的真超集和非真超集，也可理解为<和<=换了方向
#可用集合的issuperset方法检验非真超集
#issubset或issuperset能将任何可迭代参数转换为集合再操作
```

#### 集合的数学运算

```python
#并集
In [1]: {1, 3, 5} | {2, 3, 4}
Out[1]: {1, 2, 3, 4, 5}

In [2]: {1, 3, 5}.union([20, 20, 3, 40, 40])
Out[2]: {1, 3, 5, 20, 40}

#交集
In [3]: {1, 3, 5} & {2, 3, 4}
Out[3]: {3}

In [4]: {1, 3, 5}.intersection([1, 2, 2, 3, 3, 4, 4])
Out[4]: {1, 3}

#差集
In [5]: {1, 3, 5} - {2, 3, 4}
Out[5]: {1, 5}

In [6]: {1, 3, 5, 7}.difference([2, 2, 3, 3, 4, 4])
Out[6]: {1, 5, 7}

#对称差集（两个集合中互不相同的元素）
In [7]: {1, 3, 5} ^ {2, 3, 4}
Out[7]: {1, 2, 4, 5}

In [8]: {1, 3, 5, 7}.symmetric_difference{[2, 2, 3, 3, 4, 4]}
Out[8]: {1, 2, 4, 5, 7}

#不相交集（检验两个集合有无公共元素）
In [9]: {1, 3, 5}.isdisjoint({2, 4, 6})
Out[9]: True

In [10]: {1, 3, 5}.isdisjoint({4, 6, 1})
Out[10]: False
```

#### 集合的可变运算符和方法

集合也有**增强赋值运算符**，执行相应集合运算操作并重新赋值给左边的集合。

| 名称             | 运算符 | 方法                        |
| ---------------- | ------ | --------------------------- |
| 并集增强赋值     | \|=    | update                      |
| 交集增强赋值     | &=     | intersection_update         |
| 差集增强赋值     | -=     | difference_update           |
| 对称差集增强赋值 | ^=     | symmetric_difference_update |

**用于添加和删除元素的方法**

```python
In [1]: numbers = {1, 2, 3}

In [2]: numbers.add(17) #add方法将参数插入进来

In [3]: numbers.add(3) #插入已有元素时集合不变

In [4]: numbers.remove(3) #remove方法将参数从集合中移除，但参数不在集合中会引发KeyError

In [5]: numbers.discard(2) #discard方法也能移除，且参数不在集合中不会报错

In [6]: numbers.pop() #pop方法随机删除集合中一个元素且输出，若使用时集合为空会引发KeyError
Out[6]: 1

In [7]: numbers.clear() #clear方法清空集合
```

#### 集合推导式

与字典推导式一样，在花括号中定义集合推导式。

```python
In [1]: numbers = [1, 2, 2, 3, 4, 5, 6, 6, 7, 8, 9, 10, 10]

In [2]: evens = {item for item in numbers if item % 2 == 0}
#{2, 4, 6, 8, 10}
```

## 面向数组的编程

NumPy库是实现Python数组的首选的库，它提供了一种高性能的、功能丰富的n维数组类型ndarray。

### 创建数组

#### 从现有数据创建数组

array函数接收一个数组或其他元素合集作为参数并返回一个由参数元素组成的新数组。

```python
In [1]: import numpy as np #将Numpy模块作为np导入

In [2]: numbers = np.array([2, 3, 5, 7, 11])

In [3]: numbers
Out[3]: array([2, 3, 5, 7, 11])

In [4]: np.array([[1, 1, 4], [5, 1, 4]]) #多维参数创建多维数组
Out[4]: array([1, 1, 4],
              [5, 1, 4]) #NumPy自动格式化数组对齐每行的列
```

#### 用特定值填充数组

```python
In [1]: import numpy as np

In [2]: np.zeros(5) #创建全是0.的数组，默认为元素类型float64
Out[2]: array([0., 0., 0., 0., 0.])

In [3]: np.ones((2, 4), dtype = int) #创建全是1的数组，用dtype参数指定元素类型为int64
Out[3]:
array([[1, 1, 1, 1],
       [1, 1, 1, 1]])
       
In [4]: np.full((3, 5), 13) #创建以第二个参数为元素值和类型的数组
Out[4]:
array([[13, 13, 13, 13, 13],
       [13, 13, 13, 13, 13],
       [13, 13, 13, 13, 13]])
```

#### 从范围创建数组

##### 使用arange创建整数范围

arange函数使用类似内置函数range，前两个参数为范围，第三个参数为步长。

```python
In [1]: import numpy as np

In [2]: np.arange(5)
Out[2]: array([0, 1, 2, 3, 4])

In [3]: np.arange(5, 10)
Out[3]: array([5, 6, 7, 8, 9])

In [4]: np.arange(10, 1, -2)
Out[4]: array([10, 8, 6, 4, 2])
```

##### 使用linspace创建浮点范围

```python
In [5]: np.linspace(0.0, 1.0, num = 5) #创建从0.0到1.0的5个等间距元素
Out[5]: array([0. , 0.25, 0.5 , 0.75, 1. ])
```

##### 显示大数组

在显示数组时如果有1000及以上个项，NumPy会省略中间的行和列，保留最前和最后的三行和三列。

```python
In [7]: np.arange(1, 100001).reshape(100, 1000)
Out[7]:
array([[    1,     2,     3, ...,   998,   999,   1000],
       [ 1001,  1002,  1003, ...,  1998,  1999,   2000],
       [ 2001,  2002,  2003, ...,  2998,  2999,   3000],
       ...,
       [97001, 97002, 97003, ..., 97998, 97999,  98000],
       [98001, 98002, 98003, ..., 98998, 98999,  99000],
       [99001, 99002, 99003, ..., 99998, 99999, 100000]])
```

### 数组属性

本节以以下数组为例：

```python
In [1]: import numpy as np

In [2]: integers = np.array([[1, 2, 3], [4, 5, 6]])

In [3]: floats = np.array([0.0, 0.1, 0.2, 0.3, 0.4])

In [4]: floats
Out[4]: array([0., 0.1, 0.2, 0.3, 0.4]) #NumPy在浮点值中不显示小数点右侧尾随的0
```

#### 元素类型

```python
In [5]: integers.dtype #属性dtype包含数组的元素类型
Out[5]: dtype('int64') #64位整型

In [6]: floats.dtype
Out[6]: dtype('float64') #64位浮点型
```

NumPy支持的完整的类型列表间见https://docs.scipy.org/doc/numpy/user/basics.types.html

#### 维数

```python
In [7]: integers.ndim #属性ndim包含数组的维数
Out[7]: 2

In [8]: floats.ndim
Out[8]: 1

In [9]: integers.shape #属性shape包含数组每个维度上元素的个数的元组
Out[9]: (2, 3) #2行3列

In [10]: floats.shape
Out[10]: (5,) #5个
```

#### 元素数和元素大小

```python
In [11]: integers.size #属性size包含一个数组的元素总数
Out[11]: 6

In [12]: integers.itemsize #属性itemsize包含存储每个元素所需的字节数
Out[12]: 8

In [13]: floats.size
Out[13]: 5

In [14]: floats.itemsize
Out[14]: 8
```

#### 遍历多维数组

```python
In [15]: for row in integers: #以多维方式迭代数组
    ...: 	for column in row:
    ...:		print(column, end = ' ')
    ...:	print()
1 2 3 
4 5 6 

In [16]: for i in integers.flat: #用flat属性按一维形式迭代数组
    ...: 	print(i, end = ' ')
1 2 3 4 5 6 
```

### 数组运算符

#### 数组和单个数值的算术运算

```python
#单个数值的算术运算对数组中每个元素都生效
In [1]: import numpy as np

In [2]: numbers = np.arange(1, 6)

In [3]: numbers * 2
Out[3]: array([2, 4, 6, 8, 10])

In [4]: numbers ** 3
Out[4]: array([1, 8, 27, 64, 125])

#增强赋值修改左操作数的每个元素
In [5]: numbers += 10

In [6]: numbers
Out[6]: array([11, 12, 13, 14, 15])
```

#### 数组间的算术运算

```python
#相同形状的数组间执行算术运算或增强赋值，两个数组相同位置的元素各自运算得到新数组的元素
In [7]: numbers2 = np.linspace(1.1, 5.5, 5)

In [8]: numbers * numbers2
Out[8]: array([12.1, 26.4, 42.9, 61.6, 82.5])
```

#### 比较数组

```python
In [9]: numbers >= 13 #数组的每个元素分别与单个值作比较，返回一个布尔值数组
Out[9]: array([False, False, True, True, True])

In [10]: numbers2 < numbers #两个数组相同位置的元素之间作比较
Out[10]: array([True, True, True, True, True])
```

### NumPy计算方法

```python
In [1]: import numpy as np

In [2]: grades = np.array([[87, 96, 70], [100, 87, 90], [94, 77, 90], [100, 81, 82]])

In [3]: grades.sum() #sum方法计算数组所有元素的和
Out[3]: 1054

In [4]: grades.min() #min方法计算数组所有元素的最小值
Out[4]: 70

In [5]: grades.max() #max方法计算数组所有元素的最大值
Out[5]: 100

In [6]: grades.mean() #mean方法计算数组所有元素的平均值
Out[6]: 87.83333333333333

In [7]: grades.std() #std方法计算数组所有元素的标准偏差
Out[7]: 8.792357792739987

In [8]: grades.var() #var方法计算数组所有元素的方差
Out[8]: 77.30555555555556

#axis关键字参数指定在计算中使用哪个维度
In [9]: grades.mean(axis = 0) #对每个列的所有行值进行计算
Out[9]: array([95.25, 85.25, 83.  ])

In [10]: grades.mean(axis = 1) #对每个行的所有列值进行计算
Out[10]: array([84.33333333, 92.33333333, 87.        , 87.66666667])
```

NumPy数组的更多计算方法详见https://docs.scipy.org/doc/numpy/reference/arrays.ndarray.html

### 通用函数

NumPy提供许多通用函数承担算术运算符的作用。

```python
In [1]: import numpy as np

In [2]: numbers = np.array([1, 4, 9, 16, 25, 36])

In [3]: np.sqrt(numbers) #sqrt函数对数组每个元素开方
Out[3]: array([1., 2., 3., 4., 5.])

In [4]: numbers2 = np.arange(1, 7) * 10

In [5]: np.add(numbers, numbers2) #add函数等价于+运算符
Out[5]: array([11, 24, 39, 56, 75, 96])

In [6]: np.multiply(numbers2, 5) #multiply函数等价于*运算符
Out[6]: array([ 50, 100, 150, 200, 250, 300])
```

**其他通用函数**

| 分类         | 举例                                                         |
| ------------ | ------------------------------------------------------------ |
| 数学函数     | add, subtract, multiply, divide, remainder, exp, log, sqrt, power |
| 三角函数     | sin, cos, tan, hypot, arcsin, arccos, arctan                 |
| 位运算函数   | bitwise_and, bitwise_or, bitwise_xor, invert, left_shift, right_shift |
| 比较函数     | greater, greater_equal, less, less_equal, equal, not_equal, logical_and, logical_or, logical_xor, logical_not, minimum, maximum |
| 浮点运算函数 | floor, ceil, isinf, isnan, fabs, trunc                       |

完整列表详见https://docs.scipy.org/doc/numpy/reference/ufuncs.html

### 索引和切片

```python
#二维数组索引
In [1]: import numpy as np

In [2]: grades = np.array([[87, 96, 70], [100, 87, 90], [94, 77, 90], [100, 81, 82]])

In [3]: grades[0, 1] #第0行第1列
Out[3]: 96

#选择二维数组的行子集
In [4]: grades[1] #选择第1行
Out[4]: array([100, 87, 90])

In [5]: grades[0:2] #选择多个连续的行用切片表示
Out[5]:
array([[ 87,  96,  70],
       [100,  87,  90]])

In [6]: grades[[1, 3]] #选择多个非连续行用行索引列表
Out[6]:
array([[100,  87,  90],
       [100,  81,  82]])

#选择二维数组的列子集与行子集参数语法相同，逗号前选择该列中的哪些行，可以是特定的行号、一个表示行子集的切片或一个特定行索引的列表，本例“:”表示所有行的切片
In [7]: grades[:, 1:3]
Out[7]: array([[96, 70],
               [87, 90],
               [77, 90],
               [81, 82]])
```

### 视图与浅拷贝

在字典的keys和values方法中我们引入了**视图**对象，即能“看到”其他对象中的数据的对象，而不是拥有自己的数据副本。

视图是**浅拷贝**，视图与原始数据的改变是同步的。

各种数组方法和切片操作均生成数组数据的视图。

```python
In [1]: import numpy as np

In [2]: numbers = np.arange(1, 6)

In [3]: numbers2 = numbers.view() #view方法返回包含原始数组对象数据的视图的新数组

In [4]: numbers[1] *= 10

In [5]: numbers2
Out[5]: array([ 1, 20,  3,  4,  5]) #改变原始数组中的值会改变视图中的值

In [6]: numbers2[1] /= 10

In [7]: numbers
Out[7]: array([1, 2, 3, 4, 5]) #改变视图中的值也会改变原始数组中的值
```

#### 切片视图

```python
In [8]: numbers2 = numbers[0:3] #切片操作返回相应切片的视图

In [9]: numbers[1] *= 20

In [10]: numbers2
Out[10]: [ 1, 40,  3]
```

### 深拷贝

有时为了保持数据的独立性，需要使用原始数据的**深拷贝**，原始数据和拷贝数据之间不会相互影响。

```python
In [11]: numbers3 = numbers.copy() #copy方法返回原始数组对象数据的深拷贝
```

### 重塑和转置

#### reshape和resize（一维到多维）

```python
In [1]: import numpy as np

In [2]: grades = np.array([[87, 96, 70], [100, 87, 90]])

#reshape方法产生一个具有新维度的原始数组的视图
In [3]: grades.reshape(1, 6) #将一维数组转换为1行6列的二维数组
Out[3]: array([ 87,  96,  70, 100,  87,  90])

#视图和原始数组共享数据，但是保持各自的形状
In [4]: grades
Out[4]:
array([[ 87,  96,  70],
       [100,  87,  90]]) #原始数组形状不改变

#resize方法改变原始数组的形状
In [5]: grades.resize(1, 6)

In [6]: grades
Out[6]: array([ 87,  96,  70, 100,  87,  90])
```

#### flatten和ravel（多维到一维）

```python
In [7]: grades = np.array([[87, 96, 70], [100, 87, 90]])

#flatten方法创建原始数组数据的深拷贝
In [8]: flattened = grades.flatten()

In [9]: flattend
Out[9]: array([ 87,  96,  70, 100,  87,  90])

#ravel方法产生一个原始数据的视图
In [10]: raveled = grades.ravel()
```

#### 转置行和列

```python
In [11]: grades.T #T属性返回一个转置的数组视图
Out[11]:
array([[100, 100],
       [ 96,  87],
       [ 70,  90]])
```

#### 水平堆叠和垂直堆叠

可以按行或列组合数组，称为**水平堆叠**和**垂直堆叠**。

```python
In [12]: grades2 = np.array([[94, 77, 90], [100, 81, 82]])

In [13]: np.hstack((grades, grades2)) #hstack函数将元组中的数组水平堆叠起来
Out[13]:
array([[100,  96,  70,  94,  77,  90],
       [100,  87,  90, 100,  81,  82]])

In [14]: np.vstack((grades, grades2)) #vstack函数将元组中的数组垂直堆叠起来
Out[14]:
array([[100,  96,  70],
       [100,  87,  90],
       [ 94,  77,  90],
       [100,  81,  82]])
```

## 字符串

### 格式化字符串

在变量、输入输出与运算符一章我们引入了格式化字符串，在本章将深入探讨。

#### 表示类型

当为f字符串中的值指定占位符时，除非指定另一种类型，否则Python假定该值应该显示为字符串。

```python
In [1]: f'{10:d}' #d表示类型将整数值格式化为字符串
Out[1]: '10'

In [2]: f'{65:c}{97:c}' #c表示类型将整数字符代码格式化为对应的字符
Out[2]: 'A a'

In [3]: f'{"hello":s}' #s是默认的字符串表示类型，其格式化的值必须是引用字符串的变量或生成字符串的表达式
Out[3]: 'hello'

In [4]: f'{17.489:.2f}' #f表示类型将浮点数格式化为字符串，“.2”表示四舍五入保留两位小数
Out[4]: '17.49'
```

#### 字段宽度和对齐方式

```python
#表示类型前用字段宽度设置指定宽度字符串中文本出现的位置，本例设置输出宽度为10的字符串，默认右对齐
In [1]: f'{27:10d}'
Out[1]: '        27'

In [2]: f'{3.5:10f}'
Out[2]: '       3.5'

In [3]: f'{"hello":10}'
Out[3]: '     hello'

#可以使用<和>指定左对齐和右对齐，用^将值居中
In [4]: f'{27:<15d}'
Out[4]: '27             '

In [5]: f'{"hello":>15}'
Out[5]: '          hello'

In [6]: f'{3.5:^7.1f}'
Out[6]: '  3.5  '
#如果剩余奇数个字符位置，Python会将多余的空间放在右侧
```

#### 数字格式化

```python
In [1]: f'{27:+10d}' #强制显示正数符号
Out[1]: '       +27'

In [2]: f'{27:+010d}' #字符宽度前的0指定用0而不是默认的空格填充其余字符
Out[2]: '+000000027'

In [3]: f'{27:d}\n{27: d}\n{-27: d}' #在符号位值用空格替代+以对齐正负数
27
 27
-27

In [4]: f'{'12345678':,d}' #逗号将数字与千位分隔符格式化
Out[4]: '12,345,678'

In [5]: f'{'123456.78':,.2f}'
Out[5]: '123,456.78'
```

### 拼接和重复字符串

```python
In [1]: 'birth' + 'day' #+运算符拼接字符串
Out[1]: 'birthday'

In [2]: '>' * 5 #*运算符重复字符串
Out[2]: '>>>>>'
```

### 去除字符串中的空白字符

```python
In [1]: sentence = '\t \n This is a test string. \t\t \n'

In [2]: sentence.strip() #strip方法删去一个字符串开头和结尾的空白字符
Out[2]: 'This is a text string'

In [3]: sentence.lstrip() #lstrip方法只删去开头的空白字符
Out[3]: 'This is a test string. \t\t \n'

In [4]: sentence.rstrip() #rstrip方法只删去结尾的空白字符
Out[4]: '\t \n This is a test string.'
```

### 字符大小写转换

```python
In [1]: 'happy birthday'.capitalize() #capitalize方法返回只有首字母大写的新字符串
Out[1]: 'Happy birthday'

In [2]: 'strings: a deeper look'.title() #title方法返回仅大写每个单词的首字母的新字符串
Out[2]: 'Strings: A Deeper Look'
```

### 字符串的比较运算符

字符串基于字符的ASCII码值比较，字母从小到大的顺序为A-Z、a-z，可使用ord函数查看字符对应的整数。

按照字典顺序比较字符串，如'App' < 'app' < 'apple' < 'bAnana' < 'baNana'。

### 查找字符串

#### 计算子字符串出现次数

```python
In [1]: sentence = 'to be or not to be that is the question'

In [2]: sentence.count('to') #count方法计算子字符串在整个字符串中的出现次数
Out[2]: 2

In [3]: sentence.count('to', 12) #第二个参数start_index指定count方法搜索字符串切片[start_index:]
Out[3]: 1

In [4]: sentence.count('to', 12, 13) #第三个参数end_index指定字符串切片[start_index:end_index]
Out[4]: 0
```

本节其他字符串方法都有start_index和end_index参数指定搜索原始字符串的切片。

#### 定位子字符串

```python
In [5]: sentence.index('be') #index方法搜索子字符串并返回其所在的第一个索引，找不到则引起ValueError
Out[5]: 3

In [6]: sentence.rindex('be') #rindex方法从末尾搜索子字符串并返回其所在的最后一个索引，找不到则引起ValueError
Out[6]: 16
#find和rfind作用与index和rindex相同，但找不到会返回-1而不是引起ValueError
```

#### 确定是否包含子字符串

```python
In [7]: 'that' in sentence #in和not in运算符确定字符串是否包含指定的子字符串
Out[7]: True

In [8]: 'That' not in sentence
Out[8]: True
```

#### 在开头或结尾定位子字符串

```python
In [9]: sentence.startwith('to') #startwith方法判断字符串是否以指定的子字符串开始
Out[9]: True

In [10]: sentence.endwith('quest') #endwith方法判断字符串是否以指定的字符串结束
Out[10]: False
```

### 替换子字符串

```python
In [1]: values = '1\t2\t3\t4\t5'

In [2]: values.replace('\t', ',') #replace方法搜索第一个参数的字符串并用第二个参数的字符串替换，第三个参数可指定最大替换次数
Out[2]: '1,2,3,4,5'
```

### 字符串拆分和连接

#### 拆分字符串

```python
In [1]: letters = 'A, B, C, D'

#split方法将第一个参数作为定界符把字符串分成子字符串进行标记后返回标记列表，界定符默认为空白字符
In [2]: letters.split(', ')
Out[2]: ['A', 'B', 'C', 'D']

#第二个参数指定最大拆分数，最后一个标记是经最多次拆分后字符串的其余部分
In [3]: letters.split(', ', 2)
Out[3]: ['A', 'B', 'C, D']
#rsplit方法任务与split相同，但是会从末尾逆向处理给定最多次数的标记拆分
```

#### 连接字符串

```python
In [4]: letters_list = ['A', 'B', 'C', 'D']

In [5]: ','.join(letters_list) #join方法以调用该方法的字符串为分隔符将参数中的字符串拼接起来
Out[5]: 'A,B,C,D'
```

#### partition和rpartition方法

```python
#partition方法根据分隔符参数将一个字符串拆分为有三个字符串的 元组，分别是分隔符之前的字符串部分、分隔符本身和分隔符之后的字符串部分
In [7]: 'Amanda: 89, 97, 92'.partition(': ')
Out[7]: ('Amanda', ': ', '89, 97, 92')
#rpartition方法从字符串末尾搜索分隔符
```

#### splitlines方法

```python
#splitlines方法将多行文本字符串按行拆分为一个列表
In [1]: lines = """This is line1
   ...: This is line2
   ...: This is line3"""

In [2]: lines.splitlines()
Out[2]: ['This is line1', 'This is line2', 'This is line3']
#括号内如果给True参数会保留每个字符串末尾的换行符，默认为False
```

### 字符串测试方法

```python
In [1]: '-27'.isdigit() #isdigit方法检测字符串是否只包含数字字符0~9
Out[1]: False

In [2]: 'A9876'.isalnum() #isalnum方法检测字符串是否只包含数字和字母
Out[2]: True
```

| 字符串测试方法 | 描述                           |
| -------------- | ------------------------------ |
| isalnum()      | 仅包含数字和字母               |
| isalpha()      | 仅包含字母                     |
| isdecimal()    | 仅包含十进制整数且不包含正负号 |
| isdigit()      | 仅包含数字                     |
| isidentifier() | 表示有效标识符                 |
| islower()      | 字母均为小写                   |
| isnumeric()    | 表示不带正负号和小数点的数值   |
| isspace()      | 仅包含空白字符                 |
| istitle()      | 每个单词只有首字母大写         |
| isupper()      | 字母均为大写                   |

### 原始字符串

字符串的反斜杠字符会引入转义序列，要在字符串中包含反斜杠必须使用两个反斜杠字符\\\\，使得一些字符串难以阅读。

字符r开头的**原始字符串**将每个反斜杠视为普通的字符而不是转义序列的开头。

```python
In [1]: file_path = r'C:\MyFolder\MySubFolder\MyFile.txt'

In [2]: file_path
Out[2]: 'C:\\MyFolder\\MySubFolder\\MyFile.txt'
#Python仍以常规字符串的形式将反斜杠字符表示为两个反斜杠
```

### 正则表达式

**正则表达式**用于描述匹配其他字符串中字符的搜索模式。

正则表达式的作用：

* 验证文本数据
* 从文本中提取数据
* 数据清理
* 将数据转换为其他格式

#### re模块与fullmatch函数

```python
In [1]: import re #使用正则表达式需要导入re模块
```

fullmatch是最简单的正则表达式函数之一，用于检查第二个参数传入的字符串是否与第一个参数传入的模式匹配。

```python
#匹配完全相同的字符串
In [2]: pattern = '02215'

In [3]: 'Match' if re.fullmatch(pattern, '02215') else 'No match'
Out[3]: 'Match'

In [4]: 'Match' if re.fullmatch(pattern, '51220') else 'No match'
Out[4]: 'Match'
```

##### 元字符、字符类和量词

正则表达式通常包含称为**元字符**的特殊符号：[ ] { } ( ) \ * + ^ $ ? . |

元字符\作为预定义**字符类**的开始，每个字符类都能匹配一组特定的字符。

```python
#验证一个有五位数字的美国邮政编码
In [5]: 'Valid' if re.fullmatch(r'\d{5}', '02215') else 'Invalid' #元字符中反斜杠用原始字符串
Out[5]: 'Valid'

In [5]: 'Valid' if re.fullmatch(r'\d{5}', '9876') else 'Invalid'
Out[5]: 'Invalid'
#正则表达式\d{5}中，\d是表示一位数字0~9的字符类，字符类后加上一个量词{5}表示重复\d五次
```

| 字符类 | 匹配字符                                                     |
| ------ | ------------------------------------------------------------ |
| \d     | 任意数字0~9                                                  |
| \D     | 任意非数字字符                                               |
| \s     | 任意空白字符（空格、制表符、换行符）                         |
| \S     | 任意非空白字符                                               |
| \w     | 任意单词字符（也称为字母数字字符），包括任何大小写字母、数字或下划线 |
| \W     | 任意非单词字符                                               |

##### 自定义字符集

使用方括号[ ]来定义与单个字符匹配的**自定义字符类**，例如[aeiou]匹配小写元音字母，[A-Z]匹配大写字母，[a-z]匹配小写字母，[a-zA-Z]匹配任何大小写字母。

```python
#验证一个单词以大写字母开头跟随任意数量的小写字母
In [6]: 'Valid' if re.fullmatch('[A-Z][a-z]*', 'Wally') else 'Invalid' 
Out[6]: 'Valid'

In [7]: 'Valid' if re.fullmatch('[A-Z][a-z]*', 'eva') else 'Invalid'
Out[7]: 'Invalid'
#量词*匹配其左侧子表达式任意次，包括零次
#如果希望至少匹配一次则用量词+
#量词*和+都会匹配到下一个字符不匹配为止

#验证一个字符不是小写字母
In [8]: 'Valid' if re.fullmatch('^[a-z]', 'A') else 'Invalid' #^开头的自定义字符类将匹配指定范围以外的任何字符
Out[8]: 'Valid'

#自定义字符类中的元字符被视为文字字符
In [9]: 'Valid' if re.fullmatch('[*+$]', '+') else 'Invalid'
Out[9]: 'Valid'
```

##### 其他量词

```python
#?量词匹配子表达式零次或一次
#匹配label过去分词的拼写，本例?子表达式为字符'l'，匹配labeled和labelled，但不匹配labellled
In [10]: 'Match' if re.fullmatch('labll?ed', 'labeled') else 'No match' 
Out[10]: 'Match'

#{n, m}量词匹配子表达式n到m次，省略参数m则至少匹配n次
#匹配包含3~6位数字的字符串
In [11]: 'Match' if re.fullmatch('\d{3, 6}', '123') else 'No match' 
Out[11]: 'Match'

In [12]: 'Match' if re.fullmatch('\d{3, 6}', '1234567') else 'No match' 
Out[12]: 'No match'
```

#### 替换子字符串和拆分字符串

##### sub函数

sub函数在字符串中使用指定的替换文本替换模式匹配的文本。

sub函数接收三个必需参数：要匹配的模式、替换文本、要搜索的字符串，可用第四个关键字参数count指定最多替换次数。

```python
In [1]: import re

In [2]: re.sub(r'\t', ', ', '1\t2\t3\t4', count = 2) #用', '替换'1\t2\t3\t4'中的制表符'\t'两次
Out[2]: '1, 2, 3\t4'
```

##### split函数

split函数用正则表达式指定定界符来标记字符串并返回字符串列表。

```python
#定界符为逗号跟随任意数量空白字符，关键字参数maxsplit指定最多拆分数
In [3]: re.split(r',\s*', '1,  2,  3,4,   5,6,7,8', maxsplit = 3)
Out[3]: ['1', '2', '3', '4,   5,6,7,8']
```

#### 其他搜索功能、访问匹配

##### 函数search和match

search函数在字符串中查找与正则表达式匹配的第一个字符串并返回包含子字符串的匹配对象（类型为SRE_Match），无法匹配则返回None。匹配对象的group方法会返回该子字符串。

```python
In [1]: import re

In [2]: result = re.search('fun', 'Python is fun')

In [3]: result.group() if result else 'not found'
Out[3]: 'fun'

#match函数只在起始位置匹配字符串，而search函数在整个字符串中匹配

#可选flags关键字参数在匹配中忽略大小写
In [5]: result3 = re.search('Sam', 'SAM WHITE', flags = re.IGNORECASE)

In [6]: result3.group() if result else 'not found'
Out[6]: 'SAM'

#正则表达式开头^元字符表示该表达式仅匹配字符串开头
#正则表达式末尾$元字符表示该表达式仅匹配字符串末尾
In [7]: result = re.search('^Python', 'Python is fun')

In [8]: result.group() if result else 'not found'
Out[8]: 'Python'

In [7]: result = re.search('Python$', 'Python is fun')

In [8]: result.group() if result else 'not found'
Out[8]: 'Not found'
```

##### 函数findall和finditer

函数findall查找字符串中所有的匹配子字符串并返回匹配子字符串的列表。

```python
#提取所有美国电话号码
In [9]: contact = 'Wally White, Home: 555-555-1234, Work: 555-555-4321'

In [10]: re.findall(r'\d{3}-\d{3}-\d{4}', contact)
Out[10]: ['555-555-1234', '555-555-4321']
```

函数finditer功能与findall类似，但返回一个匹配对象的**惰性迭代**，一次只返回一个匹配项，可以节省内存。

```python
In [11]: for phone in re.finditer(r'\d{3}-\d{3}-\d{4}', contact)
    ...: print(phone.group())
555-555-1234
555-555-4321
```

##### 捕获匹配中的子字符串

使用**括号元字符**捕获匹配项中的子字符串。

```python
#抓取字符串文本中的姓名和电子邮件地址
In [12]: text = 'Charlie Cyan, e-mail: demo1@deitel.com'

In [13]: pattern = r'([A-Z][a-z]+ [A-Z][a-z]+), e-mail: (\w+@\w+\.\w{3})'

In [14]: result = re.research(pattern, text)

In [15]: result.groups() #匹配对象的groups方法返回抓取到的子字符串的元组
Out[15]: ('Charlie Cyan', 'demo1@deitel.com')

In [16]: result.group() #匹配对象的group方法以单个字符串形式返回整个匹配结果
Out[16]: 'Charlie Cyan, e-mail: demo1@deitel.com'

In [17]: result.group(2) #向group方法传递一个整数来访问每个抓取到的子字符串，抓取到的子字符串从1开始编号
Out[17]: 'demo1@deitel.com'
```

## 文件和异常

### 文件

Python将**文本文件**视作一个字符序列，将**二进制文件**（图像、视频等）视作一个字节序列。

文本文件的第一个字符和二进制文件的第一个字节编号为0。

### 文本文件处理

#### 写入数据

```python
In [1]: with open('accounts.txt', mode = 'w') as accounts:
   ...: 	accounts.write('100 Jones 24.98\n') #也可使用print('100 Jones 24.98', file = accounts)
   ...: 	accounts.write('200 Doe 345.67\n')
   ...: 	accounts.write('300 White 0.00\n')
   ...: 	accounts.write('400 Stone -42.16\n')
   ...: 	accounts.write('500 Rich 224.62\n')
------
#文件内容
100 Jones 24.98
200 Doe 345.67
300 White 0.00
400 Stone -42.16
500 Rich 224.62
```

**with语句**获取一个资源（accounts.txt）并将对应的对象赋给一个变量（accounts）。执行到with语句序列结尾自动调用资源对象的close方法释放资源

**open函数**打开文件（accounts.txt）并将该文件与一个文件对象相关联。mode参数指定**文件打开方式**，本例使用'w'方式打开文件以进行写文件操作，文件不存在则自动创建，未指定文件路径则在当前文件夹创建，文件已存在则自动清空内容。

文件对象的**write方法**向文件写入字符串。

#### 读取数据

```python
In [2]: with open('accounts.txt', mode = 'r') as accounts: #文件打开方式'r'以只读方式打开
   ...: 	print(f'{"Account":<10}{"Name":<10}{"Balance":>10}')
   ...: 	for record in accounts: #每次读取文件中的一行数据并以字符串形式返回
   ...: 		account, name, balance = record.split() #字符串的split方法分离各字段数据并以列表形式返回
   ...: 		print(f'{account:<10}{name:<10}{balance:>10}')
Account   Name         Balance
100       Jones          24.98
200       Doe           345.67
300       White           0.00
400       Stone         -42.96
500       Rich          224.62
```

**readlines方法**一次性读取整个文本文件并返回字符串列表，每个字符串对应一行数据，适用于小规模文件的数据读取。

读取文件时系统会维护一个**文件位置指针**指向要读取的下一个字符的位置。如果想在每次处理前将文件位置指针重新定位到文件开始位置，一种方式是关闭并重新打开文件，另一种更高效的方式是调用文件对象的**seek方法**：$\mathit{file\_object}\texttt{.seek(0)}$

**文本文件的打开模式**

| 模式 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| 'r'  | 打开文本文件进行读操作，为默认打开模式                       |
| 'w'  | 打开文本文件进行写操作，若文件已存在则内容会被自动清空       |
| 'a'  | 打开文本文件进行追加操作，若文件不存在则创建该文件，新数据会被写到文件已有数据的后面 |
| 'r+' | 打开文本文件进行读/写操作                                    |
| 'w+' | 打开文本文件进行读/写操作，若文件已存在则内容会被自动清空    |
| 'a+' | 打开文本文件进行读/追加操作，若文件不存在则创建该文件，新数据会被写到文件已有数据的后面 |

**文件对象的常用方法**

| 方法       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| read       | 返回一个字符串，包含字符/字节数量由整数参数指定，没有传入参数则返回文件全部内容 |
| write      | 向文件写入一个字符串                                         |
| readline   | 以字符串形式返回一行文本，若有换行符则返回字符串末尾包含换行符，若已到达文件结束位置则返回空字符串 |
| writelines | 接收一个字符串列表并将每个字符串按行写入文件中               |



### 更新文本文件

由于格式化的输入输出方式中记录字段数据的长度可能有所不同，在修改一个文本文件的格式化数据时可能会破坏其他数据。例如将accounts.txt中的名字White修改为Williams，如果简单地用新名字覆盖原来的名字，原记录为：

$\texttt{300 White 0.00}$

新记录为：

$\texttt{300 Williams00}$

为了使修改操作正常进行，需按以下步骤：

```python
In [1]: accounts = open('accounts.txt', 'r') #以只读方式打开原始文件

In [2]: temp_file = open('temp_file.txt', 'w') #以只写方式创建临时文件

In [3]: with accounts, temp_file:
   ...: 	for record in accounts:
   ...: 		account, name, balance = record.split() #读取并解包原始文件所有数据
   ...: 		if account != '300': #筛选非目标数据行
   ...: 			temp_file.write(record) #照抄进临时文件
   ...: 		else: #筛选目标数据行
   ...: 			new_record = ' '.join([account, 'Williams', balance]) #将名字替换后重组写入临时文件
   ...: 			temp_file.writ(new_record + '\n')
    
In [4]: import os #导入os模块以使用文件处理函数

In [5]: os.remove('accounts.txt') #删除原始文件

In [6]: os.rename('temp_file.txt', 'accounts.txt') #以原始文件名命名临时文件
```

### 使用JSON进行序列化

#### JSON数据格式

JSON对象类似于Python中的字典，每一个JSON对象对应一个用花括号括起来的由逗号分隔的属性名-属性值列表。但JSON键只能是双引号字符串，且有序、可重复。

JSON数组类似于Python中的列表，是用方括号括起来的由逗号分隔的值。

JSON对象和数组中的值可以是：

* 双引号字符串
* 数值
* JSON布尔值（true和false）
* null
* 数组
* 其他JSON对象

#### 将对象序列化为JSON

```python
In [1]: import json #导入json模块

In [2]: accounts_dict = {'accounts': [
   ...: 	{'account': 100, 'name': 'Jones', 'balance': 24.98},
   ...: 	{'account': 200, 'name': 'Doe', 'balance': 345.67}]}
#字典accounts_dict包含1个键-值对，键为accounts，值为表示两个账户信息的字典列表，每个账户的字典包含3个键-值对

In [3]: with open('accounts.json', 'w') as accounts:
   ...: 	json.dump(accounts_dict, accounts)
------
#accounts.json文件内容（经格式化处理）：
{"accounts": 
[{"account": 100, "name": "Jones", "balance": 24.98},
 {"account": 200, "name": "Doe", "balance": 345.67}]}
```

#### 反序列化JSON文本

json模块的**load函数**可以读取其文件对象参数所对应的全部JSON文本并转换为一个Python对象，即反序列化数据。

```python
In [4]: with open('accounts.json', 'r') as accounts:
   ...: 	accounts_json = json.load(accounts)

In [5]: accounts_json
Out[5]:
{'accounts': [{'account': 100, 'name': 'Jones', 'balance': 24.98}, {'account': 200, 'name': 'Doe', 'balance': 345.67}]}
```

#### 显示JSON文本

json模块的**dumps函数**将一个JSON对象以Python字符串形式返回。

dumps函数指定indent关键字参数，返回的字符串会包含换行符和良好的缩进。

```python
In [6]: with open('accounts.json', 'r') as accounts:
   ...: 	print(json.dumps(json.load(accounts), indent = 4))
{
    "accounts": [
        {
            "account": 100,
            "name": "Jones",
            "balance": 24.98
        },
        {
            "account": 200,
            "name": "Doe",
            "balance": 345.67
        }
    ]
}
```

### 处理异常

#### 被零除和无效输入

试图被零除会导致ZeroDivisionError异常。

函数收到不满足要求的参数会导致ValueError异常

#### try语句

Python使用try语句进行异常处理。

```python
#除法运算程序
while True:
    #try子句跟随一组可能引发异常的语句
    try:
        number1 = int(input('Enter numerator: ')) #输入被除数
        number2 = int(input('Enter denominator: ')) #输入除数
        result = number1 / number2 #计算结果
    #except子句指定处理的异常类型
    except ValueError: #对ValueError的处理（输入了非数字的值）
        print('You must enter two integers\n')
    except ZeroDivisionError: #对ZeroDivisionError的处理（除数输入了0）
        print('Attempted to divide by zero\n')
    #当except子句成功处理异常后程序会执行finally子句（如果有），然后再执行try语句后的代码（本例进入下一次循环）
    #可选else子句在没有任何异常的情况下执行
    else:
        print(f'{number1:.3f} / {number2:.3f} = {result:.3f}')
        break
------
#运行结果
Enter numerator: 100
Enter denominator: 0
Attempted to divide by zero

Enter numerator: 100
Enter denominator: hello
You must enter two integers

Enter numerator: 100
Enter denominator: 7
100.000 / 7.000 = 14.286
```

当一些异常类型具有相同处理代码时，以元组形式指定：

$\texttt{except (}type1\texttt{, }type2\texttt{, ...) as }variable\_name\texttt{:}$

在所有except子句和else子句（如果有）之后写一个finally子句，则finally子句必然在最后被执行到（除非程序提前终止）。

### 显式地引发一个异常

有时候我们可能需要写一个函数通过引发异常以通知调用者发生的错误，raise语句可以显式地引发一个异常，其最简单的形式是：

$\texttt{raise }ExceptionClassName$

___

**本章完**
