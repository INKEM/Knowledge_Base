> 参考书籍：《Python程序设计 人工智能案例实践》[美] 保罗·戴特尔 哈维·戴特尔 著
>
> 码字不易，求点赞收藏加关注
>
>有问题欢迎评论区讨论

@[TOC](目录)
# Python面向对象编程（三）
## 运算符重载

运算符重载用来重定义Python的运算符，使这些运算符能够支持自定义类型的对象的运算。对于每一个可重载的运算符，object类都定义了一个内置方法，如\_\_add\_\_对应加法运算符、\_\_mul\_\_对应乘法运算符。

运算符重载限制：

* 无法更改运算符的优先级。
* 无法更改运算符的左结合性或右结合性。
* 无法更改运算符的操作数个数。
* 无法创建新的运算符，只能重载已有运算符。
* 无法更改运算符处理内置类型对象的方法。
* 重载的运算符至少有一个操作数应该是自定义类的对象。

### 试用Complex类

以用来表示复数的Complex类演示运算符重载。

```python
In [1]: from complexnumber import Complex

#Complex对象的参数包括实部和虚部
In [2]: x = Complex(real = 2, imaginary = 4)

In [3]: x
Out[3]: (2 + 4i)

In [4]: y = Complex(real = 5, imaginary = -1)

In [5]: y
Out[5]: (5 - 1i)

In [6]: x + y
Out[6]: (7 + 3i)

In [7]: x += y

In [8]: x
Out[8]: (7 + 3i)
```

### Complex类的定义

```python
class Complex:
    """Complex class that represents a complex number with real and imaginary parts."""
    def __init__(self, real, imaginary):
        self.real = real
        self.imaginary = imaginary
    
    #重载+运算符，self对应左操作数，right对应右操作数
    def __add__(self, right):
        #返回参数分别为实部相加和虚部相加的Complex对象
        return Complex(self.real + right.real, self.imaginary + right.imaginary)
    
    #重载+=增强赋值运算符
    def __iadd__(self, right):
        #将self的实部和虚部分别增强赋值后返回
        self.real += right.real
        self.imaginary += right.imaginary
        return self
    
    #Complex对象的字符串表示
    def __repr__(self):
        return (f'({self.real}' +
                ('+' if self.imaginary >= 0 else '-') +
                f'{abs(self.imaginary)}i)')
```

## 异常类层次结构和自定义异常

每个异常都是Python异常类层次结构中某个类的对象或者是已有异常类的派生类的对象。异常类都是以BaseException作为基类、通过直接或间接继承它而得到，它们在exceptions模块中定义。

Python定义了四个重要的BaseException子类：

* SystemExit：终止程序执行（或终止一个交互式会话）时发生，未被捕获时不会像其他异常类型一样产生回溯信息。
* KeyboardInterrupt：用户在大多数系统中键入中断命令Ctrl+C时发生。
* GeneratorExit：生成器关闭（通常是生成器完成生成值的操作或其close方法被显示调用）时发生。
* Exception：大多数异常的基类，例如ZeroDivisionError、NameError、ValueError等。

异常类层次结构的一个优点是异常处理程序可以捕获特定类型的异常，或者可以使用基类类型来捕获基类异常和所有相关的子类异常。

从代码中引发异常时通常应该使用Python标准库中的现有异常类，使用继承从Exception类直接或间接派生也可以创建自己的自定义异常类，仅当需要以不同于其他已有异常类型的方式捕获和处理异常时才定义新的异常类，而这种情况是很少见的。

## 数据类简介

具名元组允许通过名字引用其成员，但它们仍只是元组而不是类。通过Python标准库的dataclasses模块可以使用兼具了具名元组优点以及传统Python类功能的数据类。

数据类能够自动生成数据属性以及\_\_init\_\_和\_\_repr\_\_方法，也会自动生成\_\_eq\_\_方法用于重载==运算符，该方法也隐式支持!=运算符。

### 创建Card数据类

以表示扑克牌的Card类为例创建数据类：

```python
from dataclasses import dataclass
from typing import ClassVar, List
```

使用$\texttt{@dataclass}$装饰器将类指定为数据类，其后加括号并指定包含的参数可以帮助数据类确定要自动生成的方法，例如$\texttt{@dataclass(order=True)}$将使数据类为<、<=、>和>=自动生成比较运算符的重载方法。

```python
@dataclass
class Card:
```

数据类在类中但在类的方法之外通过**变量注释**声明类属性和数据属性，下方代码块中$\texttt{: ClassVar[List[str]]}$指定$\texttt{FACES}$和$\texttt{SUITS}$是一个引用字符串列表（$\texttt{List[str]}$）的类属性（$\texttt{ClassVar}$），$\texttt{:str}$表示每一个数据属性都应该引用一个字符串对象。

```python
    FACES: ClassVar[List[str]] = ['Ace', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'Jack', 'Queen', 'King']
    SUITS: ClassVar[List[str]] = ['Hearts', 'Diamonds', 'Clubs', 'Spades']
    face: str
    suit: str
```

数据类property和其他方法的定义与传统类相同。

```python
	@property
    #返回扑克牌的png文件名
    def image_name(self):
        return str(self).replace(' ', '_') + '.png'
    
    #返回'face of suit'格式的字符串
    def __str__(self):
        return f'{self.face} of {self.suit}'
    
    #__format__方法在Card对象被格式化为字符串时被调用
    def __format__(self, format):
        return f'{str(self):{format}}'
```

### 数据类相对于传统类的优势

* 能自动生成\_\_init\_\_、\_\_repr\_\_等方法，节省开发时间。
* 能自动生成用于重载<、<=、>和>=这些比较运算符的特殊方法。
* 当更改数据类中定义的数据属性，然后在脚本或交互式会话中使用它时，自动生成的代码会自动更新，需要维护和调试的代码量会更少。
* 通过类属性和数据属性所需的变量注释，可以更好地利用静态代码分析工具，在代码被执行前根据静态分析结果消除一些错误。

___

**本章完**
