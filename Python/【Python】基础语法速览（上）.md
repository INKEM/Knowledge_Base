> 本文力图用最快的方式向大家陈列Python的基础语法，适合接触过其他编程语言后快速上手Python或供查阅巩固用
> 
> 参考书籍：《Python程序设计 人工智能案例实践》[美] 保罗·戴特尔 哈维·戴特尔 著
>
> 码字不易，求点赞收藏加关注
>
>有问题欢迎评论区讨论

@[TOC](目录)
下篇将介绍字典和集合、Numpy面向数组编程、字符串、文件和异常
面向对象编程将单独出一期
___
# Python基础语法速览（上）
___
## 变量、输入输出与运算符

### 变量和赋值语句

```python
In [1]: x = 7 #创建变量x并为x赋值为7
```

**变量名**是一个**标识符**，由字母、数字和下划线组成，但不能以数字开头，区分大小写

```python
In [2]: type(x) #查看x的数据类型
Out[2]: int
```

### 运算符

#### 算术运算符

| Python运算           | 算术运算符 |
| -------------------- | ---------- |
| 加法                 | +          |
| 减法                 | -          |
| 乘法                 | *          |
| 幂                   | **         |
| 除法（结果为浮点数） | /          |
| 整除                 | //         |
| 取余                 | %          |

**运算符优先级规则**

1.从最内层计算括号中的表达式

2.幂运算，多个幂运算按照从右到左的顺序计算

3.乘法、除法、整除和模，多个乘法、除法、整除和模按照从左到右的顺序计算

4.加法、减法，多个加法、减法按照从左到右的顺序计算

#### 比较运算符

**条件**是一个值为True或False的布尔表达式

| 比较运算符 | 含义     |
| ---------- | -------- |
| >          | 大于     |
| <          | 小于     |
| >=         | 大于等于 |
| <=         | 小于等于 |
| ==         | 等于     |
| !=         | 不等于   |

```python
In [1]: 114 > 514
Out[1]: False

In [2]: 114 < 514
Out[2]: True

In [3]: x = 191

In [4]: 114 <= x <= 514 #链式比较检测一个值是否在某个范围内
Out[4]: True 
```

对其他表达式，非零值为True，零为False，非空字符串为True，空字符串为False

#### 布尔运算符

条件运算符可用于组成简单条件，要将简单条件进行组合构成更复杂的条件可以使用布尔运算符and、or和not将两个表达式连接起来

**布尔运算符and**

| 表达式1 | 表达式2 | 表达式1 and 表达式2 |
| ------- | ------- | ------------------- |
| False   | False   | False               |
| False   | True    | False               |
| True    | False   | False               |
| True    | True    | True                |

**布尔运算符or**

| 表达式1 | 表达式2 | 表达式1 and 表达式2 |
| ------- | ------- | ------------------- |
| False   | False   | False               |
| False   | True    | True                |
| True    | False   | True                |
| True    | True    | True                |

**布尔运算符not**

| 表达式 | not 表达式 |
| ------ | ---------- |
| False  | True       |
| True   | False      |

**运算符优先级和结合性汇总**

| 运算符       | 结合性   |
| ------------ | -------- |
| ()           | 从左到右 |
| **           | 从右到左 |
| *  /  //  %  | 从左到右 |
| +  =         | 从左到右 |
| <  <=  >  >= | 从左到右 |
| ==  !=       | 从左到右 |
| not          | 从左到右 |
| and          | 从左到右 |
| or           | 从左到右 |

### 增强赋值

当相同的变量名同时出现在赋值运算符的左右两端，可以使用增强赋值对赋值表达式进行缩写

| 增强赋值表达式 | 解释       |
| -------------- | ---------- |
| a += 2         | a = a + 2  |
| a -= 2         | a = a - 2  |
| a *= 2         | a = a * 2  |
| a **= 2        | a = a ** 2 |
| a /= 2         | a = a / 2  |
| a //= 2        | a = a // 2 |
| a %= 2         | a = a % 2  |

### print函数、单双引号、转义字符

```python
In [1]: print('Welcome to Python!') #将括号中的参数显示为一行文本
Welcome to Python!
```

除了单引号，双引号也可以括起一个字符串，但一般习惯用单引号

```python
In [2]: print('Welcome', 'to', 'Python!') #逗号分隔参数，输出自动加空格
Welcome to Python!
```

反斜杠（\）称为**转义字符**，反斜杠和紧随其后的字符形成一个**转义序列**，例如转义序列“\n”表示**换行符**

```python
In [3]: print('Welcome\nto\n\nPython!')
Welcome
to

Python!
```

| 转义序列 | 说明       |
| -------- | ---------- |
| \n       | 换行符     |
| \t       | 制表符     |
| \\\      | 插入反斜杠 |
| \\"      | 插入双引号 |
| \\'      | 插入单引号 |

在一行的结尾用续行符“\”将一个长字符串写成多行

```python
In [4]: print('this is a longer string, so we\
   ...: split it over two lines')
this is a longer string, so we split it over two lines
```

可以在print语句中执行计算

```python
In [5]: print('Sum is', 7 + 3)
Sum is 10
```

### 三引号字符串

**字符串中包含引号**

单引号字符串中可以包含双引号，双引号字符串中可以包含单引号，但单引号包含单引号和双引号包含双引号需要用转义字符

三引号字符串可以将单引号和双引号都包含在内

**多行字符串**

```python
In [1]: triple_quoted_string = """This is a triple-quoted
   ...: string that spans two lines""" #用引号将字符串赋给变量，三引号中可直接用回车键代替换行符

In [2]: print(triple_quoted_string)
This is a triple-quoted
string that spans two lines

In [3]: triple_quoted_string
Out[3]: 'This is a triple-quoted\nstring that spans two lines' #变量嵌入换行符存储多行字符串
```

### 格式化字符串

字符串引号前加字母f可以将变量用花括号括起来插入字符串来格式化输出结果

```python
In [1]: a = 1.14514

In [2]: print(f'The number a is {a}')
Out[2]: The number a is 1.14514
```

更多格式化字符串内容在字符串章节讨论

### 获取输入

```python
In [1]: name = input("What's your name? ") #input函数显示字符串参数作为提示后等待并返回用户输入，随后赋值给变量
What's your name? Paul

In [2]: value = int(input('Enter an integer: ')) #input函数只会将输入转换为字符串，需要用int函数将字符串转换为整数，同理float函数可以将字符串转换为浮点数
Enter an integer: 114514

In [3]: int(114.514) #int函数还可以将小数向下取整
Out[3]: 114
```

## 控制语句

### if语句

**if语句**根据条件来决定是否执行一条语句

```python
if 114 > 514:
	print("Not homo")
if 114 < 514:
	print("Homo")
------
Homo
```

（分割线以下为程序执行后部分）

**if...else语句**包含满足条件执行的语句和不满足条件执行的雨具

```python
#成绩合格判断程序
grade = 57
if grade >= 60:
	print('Passed')
else:
	print('Failed')
------
Failed
```

**if...elif...else语句**在多种条件中选择要执行的语句

```python
#成绩等级判断程序
grade = 77
if grade >= 90:
	print('A')
elif grade >= 80:
	print('B')
elif grade >= 70:
	print('C')
elif grade >= 60:
	print('D')
else:
	print('F')
------
C
```

else非必需，即不满足任何一种条件时不执行任何语句

### while语句

**while语句**在循环条件为True时重复执行内部语句

```python
#寻找第一个大于114的2的幂
a = 2
while a <= 114:
	a = a * 2
print(a)
------
128
```

### for语句

**for语句**为一个序列中的每一项重复执行内部语句

```python
#输出'Programming'中的每个字母并用空格隔开
for character in 'Programming':
	print(character, end = ' ')
------
P r o g r a m m i n g 
```

执行步骤：

1.进入循环语句，将"Programming"中的第一个字母P赋值给character

2.执行循环体中的语句

3.将下一个字母赋值给character并执行循环体中的语句，直到所有字母都被处理过

除了字符串，还有其他可迭代的对象序列类型

**列表**是用方括号括起来并用逗号分隔的项的合集

```python
#求列表内所有项的和
total = 0
for number in [1, 1, 4, 5, 1, 4]:
	total = total + number
print(total)
------
16
```

列表也可以赋值给一个变量，用变量名替代

#### 内置函数range

```python
#单参数：创建一个从0开始一直到（但不包括）参数值的整数序列
for counter in range(10):
	print(counter, end = ' ')
------
0 1 2 3 4 5 6 7 8 9 
------
#双参数：创建一个从第一个参数开始一直到（但不包括）第二个参数的整数序列
for counter in range(5, 10):
	print(counter, end = ' ')
------
5 6 7 8 9 
------
#三参数：创建一个从第一个参数开始一直到（但不包括）第二个参数，并以第三个参数值（步长）递增的整数序列
for counter in range(1, 14, 3):
	print(counter, end = ' ')
------
1 4 7 10 13
------
# 如果第三个参数为负则递减
```

### break和continue语句

**break和continue语句**用在循环体内改变循环的控制流，执行break语句则立即退出该循环，在while语句中执行continue语句会转回循环条件以确定循环是否该继续执行，在for语句中执行continue语句会直接处理序列中的下一项（如果有）

```python
#break语句
for number in range(100):
	if number == 10:
		break
	print(number, end = ' ')
------
0 1 2 3 4 5 6 7 8 9 
------
#continue语句
for number in range(10):
	if number == 5:
		continue
	print(number, end = ' ')
------
0 1 2 3 4 6 7 8 9 
```

## 函数

### 函数定义

函数执行一项明确定义的任务，定义了一个函数后，可以在整个程序中多次调用这个函数

```python
#定义一个求平方的函数
def square(number): #def 函数名(参数1, 参数2, ...):
	return number ** 2 #将结果返回给调用者
printf(square(7)) #函数名(参数1, 参数2, ...)，参数数量与定义的数量一致
------
49
```

如果函数不需要参数，则定义和调用时括号内为空

如果return语句不带表达式将返回None（空值），在条件语句中被判断为False

如果没有return语句函数将在执行到语句块内最后一条语句后返回None

在函数的语句块中定义的参数和变量为**局部变量**，只能在函数体内部使用，在外部需要另外定义

#### 默认参数值

```python
In [1]: def rectangle_area(length = 2, width = 3): #指定参数具有默认值
   ...: 	return length * width

In [2]: rectangle_area() #调用函数时没有参数，函数使用默认值
Out[2]: 6

In [3]: rectangle_area(10) #调用函数时有一部分参数，函数从左往右给参数赋值，其余保留默认值
Out[3]: 30
```

#### 关键字参数

```python
In [1]: def rectangle_area(length, width): 
   ...: 	return length * width

In [2]: rectangle_area(width = 5, length = 10) #调用函数使用关键字参数能以任何顺序传递参数
Out[2]: 50
```

#### 不定长参数列表

```python
def average(*args): #*将参数打包成元组传递给参数args
	return sum(args)/len(args) #sum为内置求和函数，len为内置序列长度函数
grades = [88, 75, 96, 55, 83]
print(average(*grades)) #*将参数解包，此处调用等同于average(88, 75, 96, 55, 83)
------
79.4
```

**内置函数max和min**是不定长参数函数，可分别用于求出各自参数中的最大值和最小值

```python
In [1]: max(11, 45, 14)
Out[1]: 45

In [2]: min(11, 45, 14)
Out[2]: 11
```

#### 作用域

在函数体内定义的变量为**局部变量**，作用域为函数内部

在函数体外定义的变量为**全局变量**，作用域在整个程序

函数体可以直接访问全局变量的值，但无法修改，如果在函数体为全局变量赋值会创建一个同名的新局部变量

使用**global语句**在函数体中修改全局变量

```python
x = 'goodbye'
def modify_global():
	global x
	x = 'hello'
	print('x printed from modify_global:', x)
modify_global()
------
x printed from modify_global: hello
```

#### lambda函数

对于功能简单只返回一个单一表达式值的函数，可以在调用函数的位置使用一个lambda表达式临时定义一个函数

```python
def is_odd(x):
	return x % 2 != 0

#用lambda定义
lambda x: x % 2 != 0
```

### Python标准库

在编写程序时经常会用到Python标准库或其他库中的函数和类，模块是Python标准库中对相互关联的函数、数据和类进行分组的文件

> **一些常用的Python标准库模块**
>
> collections：列表、元组、字典和集合之外的数据结构
>
> Cryptography：加密数据以实现安全传输
>
> csv：处理用逗号分隔值的文件（如Excel）
>
> datetime：日期和实践操作
>
> decimal：定点和浮点算术运算，包括货币计算
>
> doctest：在简单单元测试的文档字符串中嵌入验证测试和预期结果
>
> gettext和locale：国际化和本地化模块
>
> json：与Web服务和NoSQL文档数据库一起使用的JSON处理
>
> math：常见的数学常数和操作
>
> os：与操作系统交互
>
> profile、pstats、timeit：性能分析
>
> random：伪随机数
>
> re：用于模式匹配的正则表达式
>
> sqlite3：SQLite关系数据库访问
>
> statistics：数学统计函数
>
> string：字符串处理
>
> sys：命令行参数处理：标准输入、标准输出和标准错误流
>
> tkinter：图形用户界面和基于画布的图形
>
> turtle：海龟图
>
> webbrowser：用于在Python应用程序中方便地显示网页

#### 随机数生成

Python标准库的random模块可以模拟偶然因素

```python
import random #导入random模块
random.seed(32) #设置随机数生成器的种子
for roll in range(10):
	print(random.randrange(1, 7), end = ' ') #randrange函数随机生成一个从第一个参数到（但不包括）第二个参数之间的整数值
------
1 2 2 3 6 2 4 1 6 1 
```

函数randrange生成的是基于以一个称为seed的数值开头的内部计算生成的伪随机数，每次新的会话或执行含随机函数的脚本Python会使用不同的seed，可以用seed函数为随机数生成器设置种子，相同种子的伪随机数序列相同，便于程序调试

#### math模块中的函数

| 函数                             | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| ceil(x)                          | 将x向上取整                                                  |
| floor(x)                         | 将x向下取整                                                  |
| sin(x)<br />cos(x)<br />tan(x)   | 求x的正弦<br />求x的余弦<br />求x的正切（以上均为弧度制）    |
| exp(x)<br />log(x)<br />log10(x) | 指数函数e^x^<br />求x的自然对数（底为e）<br />求x的对数（底为10） |
| pow(x, y)<br />sqrt(x)           | 求x的y次幂<br />求x的平方根                                  |
| fabs(x)                          | 求x的绝对值，返回浮点数。Python内置abs绝对值函数根据参数返回整数或浮点数 |
| fmod(x, y)                       | x除以y的余数，返回浮点数                                     |

#### import语句

```python
#用import 模块名导入，需要通过模块名称和一个点（.）访问包含在模块内的函数
In [1]: import math

In [2]: math.ceil(10.3)
Out[2]: 11
#用from 模块名 import 函数1, 函数2, ...导入，可以直接使用函数名
In [3]: from math import ceil, floor

In [4]: ceil(10.3)
Out[4]: 11
#“from 模块名 import *”使用通配符*导入模块中的所有函数，但可能触发不容易察觉的错误，如变量名和函数名歧义
#“import 模块名 as 自定义模块名”可以用自定义缩写来表示导入的模块来简化代码
```

## 序列

### 列表

列表通常存储**同构数据**（数据类型相同），也可以存储**异构数据**（数据类型不同），元素和长度可以修改

```python
In [1]: c = [-45, 6, 0, 72, 114] #创建列表

In [2]: c[0] #访问列表中第n+1个元素，因为列表中元素编号从0开始
Out[2]:-45

In [3]: len(c) #获取列表的长度
Out[3]: 5

In [4]: c[-1] #用负数访问列表，列表负数编号最后一个元素为-1，从后往前递减
Out[4]: 114

In [5]: c[2] = 514 #修改列表中的元素，此时c为[-45, 6, 514, 72, 114]，而字符串和元组序列无法修改元素

In [6]: c += [1919] #创建一个单元素列表并添加到列表c末尾，此时c为[-45, 6, 514, 72, 114, 1919]

In [7]: ho = [1, 1, 4]

In [8]: mo = [5, 1, 4]

In [9]: homo = ho + mo #运算符+拼接两个列表，此时homo为[1, 1, 4, 5, 1, 4]

In [10]: homo *= 2 #成倍地扩充序列，此时homo为[1, 1, 4, 5, 1, 4, 1, 1, 4, 5, 1, 4]
```

比较运算符逐一比较列表中的每个元素

```python
In [10]: a = [1, 2, 3]

In [11]: b = [1, 2, 3]

In [12]: c = [1, 2, 3, 4]

In [13]: a == b
Out[13]: True #a和b每个元素都相等

In [15]: a < c
Out[14]: True #a的元素数量比c少
```

#### 列表处理方法

##### sort方法（排序）

```python
In [1]: numbers = [10, 3, 7, 1, 9, 4, 2, 8, 5, 6]

In [2]: numbers.sort() #按升序排列列表元素[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

In [3]: numbers.sort(reverse = True) #按降序排列列表元素（默认值为False）

In [4]: numbers = [10, 3, 7, 1, 9, 4, 2, 8, 5, 6]

In [5]: ascending_numbers = sorted(numbers) #内置函数sorted返回按升序排列好的列表给新列表，不改变原列表
```

字符和字符串元素将按ASCII表顺序排列

##### insert方法（插入）

```python
In [1]: color_names = ['orange', 'yellow', 'green']

In [2]: color_names.insert[0, red]
#在索引0处插入'red'：['red', 'orange', 'yellow', 'green']
```

##### append方法（末尾添加）

```python
In [3]: color_names.append('blue')
#在列表末尾添加'blue'：['red', 'orange', 'yellow', 'green', 'blue']
```

##### extend方法（末尾扩充）

```python
In [4]: color_names.extend(['indigo', 'violet'])
#将另一个序列的所有元素添加到列表的末尾：['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet']
#等效于+=
```

##### remove方法（移除）

```python
In [5]: color_names.remove('green')
#移除列表中某个值的第一个匹配项：['red', 'orange', 'yellow', 'blue', 'indigo', 'violet']
```

##### clear方法（清空）

```python
In [6]: color_names.clear() #清空列表中的所有元素
```

##### count方法（计数）

```python
In [7]: responses = [1, 2, 5, 4, 3, 5, 2, 1, 3, 3, 1, 4, 3, 3, 3, 2, 3, 3, 2, 2]

In [8]: responses.count(3) #返回某个元素在列表中出现的次数
Out[8]: 8
```

##### reverse方法（反转）

```python
In [9]: color_names = ['red', 'orange', 'yellow', 'green', 'blue']

In [10]: color_names.reverse()
#反转列表中的元素：['blue', 'green', 'yellow', 'orange', 'red']
```

##### copy方法（拷贝）

```python
In [11]: copied_list = color_names.copy() #返回一个包含原始对象浅拷贝的新列表

In [12]: copied_list
Out[12]: ['blue', 'green', 'yellow', 'orange', 'red']
```

#### 列表推导式

**列表推导式**是将一个列表转换成另一个列表的工具，在转换过程中可以指定元素必须符合一定的条件才能添加到新的列表中

```python
In [1]: list1 = [item for item in range(1, 6)]
#创建整数列表：[1, 2, 3, 4, 5]

In [2]: list2 = [item ** 3 for item in range(1, 6)]
#将每个值的立方映射到新列表：[1, 8, 27, 64, 125]

In [3]: list3 = [item for item in range(1, 11) if item % 2 == 0]
#过滤出偶数到新列表：[2, 4, 6, 8, 10]

In [4]: colors = ['red', 'orange', 'yellow', 'green', 'blue']

In [5]: colors2 = [item.upper() for item in colors]
#将所有元素大写映射到新列表：['RED', 'ORANGE', 'YELLOW', 'GREEN', 'BLUE']
#总结：for前映射操作，if后过滤条件
```

#### 生成器表达式

列表推导式使用**贪婪计算**，每次都会创建一个包含了所有值的列表，用[]括起来

生成器表达式使用**惰性计算**，只返回符合要求的值，用()括起来

```python
numbers = [10, 3, 7, 1, 9, 4, 2, 8, 5, 6]
for value in (x ** 2 for x in numbers if x % 2 != 0):
	print(value, end = ' ')
------
9 49 1 81 25
```

#### 二维列表

用两个索引来表示元素的列表称为**二维列表**

```python
In [1]: a = [[77, 68, 86, 73], [96, 87, 89, 81], [70, 90, 86, 81]]
#创建一个三行四列的二维列表，每个二级方括号为一行

In [2]: a[0][2] #访问行索引为0，列索引为2的元素
Out[2]: 86

In [3]: for row in a:
   ...: 	for item in row:
   ...: 		print(item, end = ' ')
   			print()
77 68 86 73
96 87 89 81
70 90 86 81
#用嵌套for语句按行输出二维列表，可见二维列表先按行提取再按列提取
```

### 元组

元组通常存储异构数据，也可以存储同构数据，元素和长度不能随意更改，只能给整个元组重新赋值

```python
In [1]: student_tuple = () #创建一个空元组

In [2]: student_tuple = 'John', 'Green', 3.3 #用逗号分隔构造元组，括号可选

In [3]: student_tuple = ('red',) #用逗号和括号构造单元素元组

In [4]: time_tuple = (1919, 8, 10)

In [5]: time_tuple[0] #访问元组元素与列表同理
Out[5]: 1919

In [6]: time_tuple += (1, 2) #将元组time_tuple拼接上元组(1, 2)再重新赋值给time_tuple

In [7]: numbers = [1, 2, 3]

In [8]: numbers += (4, 5) #将元组附加到列表中，此时numbers为[1, 2, 3, 4, 5]

In [9]: tuple = ('ho', 'mo', [11, 45, 14])

In [10]: tuple[2][1] = 54 #元组不可变，但元组中列表的元素可变，此时tuple玩('ho', 'mo', [11, 54, 14])
```

### 序列解包

```python
In [1]: student_tuple = ('Amanda', 114)

In [2]: name, grades = student_tuple #序列可以分配给用逗号分隔的变量列表，此时name为'Aman'，grades为114

In [3]: number1 = 114

In [4]: number2 = 514

In [5]: number1, number2 = (number2, number1) #用打包和解包来交换两个变量的值
```

**内置函数enumerate**对列表中每个元素返回一个索引和值的元组

```python
In [6]: colors = ['red', 'orange', 'yellow']

In [7]: list(enumerate(colors)) #内置函数list创建一个列表包含enumerate的输出
Out[7]: [(0, 'red'), (1, 'orange'), (2, 'yellow')]

In [8]: tuple(enumerate(colors)) #同理内置函数tuple从序列创建一个元组
Out[8]: ((0, 'red'), (1, 'orange'), (2, 'yellow'))
```

### 序列切片

```python
In [1]: numbers = [2, 3, 5, 7, 11, 13, 17, 19]

In [2]: numbers[2:6] #从第一个想要的对象开始到第一个不想要的对象结束的切片
Out[2]: [5, 7, 11, 13]

In [3]: numbers[:6] #省略起始索引默认从0开始
Out[3]: [2, 3, 5, 7, 11, 13]

In [4]: numbers[6:] #省略结束索引默认到序列末尾结束
Out[4]: [17, 19]

In [5]: numbers[:] #省略开始索引和结束索引会复制整个序列
Out[5]: [2, 3, 5, 7, 11, 13, 17, 19]
```

#### 切片的步长

```python
In [6]: numbers[::2] #以2为步长构造间隔1个元素的切片
Out[6]: [2, 5, 11, 17]

In [7]: numbers[::-1] #负数步长以倒序构造切片，该行代码等价于numbers[-1:-9:-1]
Out[7]: [19, 17, 13, 11, 7, 5, 3, 2]
```

#### 切片修改列表

```python
In [8]: numbers[0:3] = ['two', 'three', 'five'] #将numbers前三个元素替换

In [9]: numbers[0:3] = [] #将numbers前三个元素删除

In [10]: numbers = [2, 3, 5, 7, 11, 13, 17, 19]

In [11]: numbers[::2] = [100, 100, 100, 100] #间隔1个元素赋值列表元素

In [12]: numbers
Out[12]: [100, 3, 100, 7, 100, 13, 100, 19]
```

#### del声明

```python
In [1]: numbers = list(range(0, 10)) #创建列表[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

In [2]: del numbers[-1] #删除列表最后一个元素[0, 1, 2, 3, 4, 5, 6, 7, 8]

In [3]: del numbers[0:2] #删除列表前两个元素[2, 3, 4, 5, 6, 7, 8]

In [4]: del numbers[::2] #每隔一个步长删除列表中的一个元素[3, 5, 7]

In [5]: del numbers[:] #删除列表的所有元素

In [6]: del numbers #删除numbers这个变量
```

### 序列搜索

#### index方法

```python
In [1]: numbers = [3, 7, 1, 4, 2, 8, 5, 6]

In [2]: numbers.index(5) #从索引0开始搜索列表，返回与关键词匹配的第一个元素的索引
Out[2]: 6

In [3]: numbers *= 2

In [4]: numbers.index(5, 7) #从索引7到列表末尾的所有元素中搜索5
Out[4]: 14

In [5]: numbers.index(7, 0, 4) #在索引0到3范围内查找值等于7的元素
Out[5]: 1
```

#### in和not in

```python
In [6]: 1000 in numbers #检测1000是否在序列里
Out[6]: False

In [7]: 1000 not in numbers #检测1000是否不在序列里
Out[7]: True
```

### 序列处理函数

#### filter函数（过滤）

```python
In [1]: numbers = [10, 3, 7, 1, 9, 4, 2, 8, 5, 6]

In [2]: list(filter(lambda x: x % 2 != 0, numbers)) #过滤出序列numbers中使得is_odd为真的元素
Out[2]: [3, 7, 1, 9, 5]
```

#### map函数（映射）

```python
In [3]: list(map(lambda x: x ** 2, numbers)) #将序列中元素平方处理后映射到新列表
Out[3]: [100, 9, 49, 1, 81, 16, 4, 64, 25, 36]
```

#### 归约

内置函数len（求长度）、sum（求和）、min（求最小值）和max（求最大值）将序列的元素处理为单个值，称为**归约**

#### key函数（查找最值）

归约函数min和max使用数字列表作为参数，在更复杂的对象比如字符串中找最值需要使用key函数

```python
In [1]: colors = ['Red', 'orange', 'Yellow', 'green', 'Blue']

In [2]: min(colors, key = lambda s: s.lower())
Out[2]: 'Blue'
#key的参数调用一个返回值的单参数函数对序列中元素进行处理再在min函数中作比较，因为字符串比较用的ASCII值小写字母比大写字母大，所以按字母表顺序需要用lower方法将所有字符串统一处理为小写字母
```

#### reverse函数（反向迭代）

```python
In [1]: numbers = [10, 3, 7, 1, 9, 4, 2, 8, 5, 6]

In [2]: reversed_numbers = [item for item in reversed(numbers)]
#反向迭代序列numbers的值，此时reversed_numbers为[6, 5, 8, 2, 4, 9, 1, 7, 3, 10]
```

#### zip函数（合并）

zip函数同时遍历多个可迭代对象的数据并把相同索引的元素提取出来打包成元组返回

```python
names = ['Bob', 'Sue', 'Amanda']
grade_point_averages = [3.5, 4.0, 3.75]
for name, gpa in zip(names, grade_point_averages):
	printf(f'Name={name}; GPA={gpa}')
------
Name=Bob; GPA=3.5
Name=Sue; GPA=4.0
Name=Amanda; GPA=3.75
```

**未完待续**
