> 参考书籍：《Python程序设计 人工智能案例实践》[美] 保罗·戴特尔 哈维·戴特尔 著
>
> 码字不易，求点赞收藏加关注
>
>有问题欢迎评论区讨论

@[TOC](目录)
# Python面向对象编程（一）

## 面向对象编程简介

Python中每一个数据都是一个**对象**。正如房屋是照蓝图建造的一样，对象是用**类**创建的，创建自定义类就是创建一种新的数据类型。类是面向对象编程的核心技术之一，创建有价值的类对构建满足需求的应用程序会有所帮助。

在创建新类时，可以通过**继承**已定义**基类**（也称作**父类**）的属性（类版本的变量）和方法（类版本的函数）指定新类的初始组成，新类被称为**派生类**（或**子类**）。继承后，还可以自定义派生类以满足应用程序的具体需求。

**多态**可以简单地将同样的方法调用发送到可能是许多不同类型的对象，每一个对象都会根据对象所属类型决定执行哪个类的方法，因此同样的方法调用可能产生多种不同的处理。

## 自定义Account类

我们以银行的Account（账户）类开始本节内容，这里考虑的Account类仅包括持有者姓名和余额这两个信息。

### 试用Account类

在定义Account类之前，先了解一下Account类的功能。

```python
In [1]: from account import Account #导入Account类

In [2]: from decimal import Decimal #Account类以Decimal类型保存和操作账户余额，因此导入Decimal类

#类的对象使用类名+括号创建，被称为构造器表达式，构造器表达式创建新对象，并利用小括号中指定的参数初始化新对象中的数据
In [3]: account1 = Account('John Green', Decimal('50.00'))

#访问Account对象的name和balance属性
In [4]: account1.name
Out[4]: 'John Green'

In [5]: account1.balance
Out[5]: Decimal('50.00')

#Account的deposit方法接收一个正数美元金额并将其加到余额上，传入错误参数导致ValueError异常
In [6]: acount1.deposit(Decimal('25.53'))
#此时account1.balance为75.53
```

### 定义Account类

一个类的定义以class关键字开始，后面跟着类名和冒号，这行代码被称为类头。紧跟着类头通常提供一个描述性的文档字符串。

```python
from decimal import Decimal

class Account:
    """Account class for maintaining a bank account balance.""" 
```

#### __init__方法

构造器表达式创建新对象通过调用类的\_\_init\_\_方法初始化对象数据，每个新类都可以提供一个\_\_init\_\_方法指定一个对象的数据属性的初始化方式。

当指定对象调用一个方法时，Python隐式地将这个对象的引用传递给所调用的方法并作为这个方法的第一个参数，因此一个类中的每个方法都必须至少包含一个参数，按照惯例接收对象引用的第一个参数被命名为self。

```python
    def __init__(self, name, balance):
        
        #验证balance参数是否有效，若为负数则返回一个ValueError异常
    	if balance < Decimal('0.00'):
        	raise ValueError('Initial balance must be >= to 0.00.')
    	self.name = name #为对象添加属性
    	self.balance = balance
```

Python类可以定义许多特殊方法，如\_\_init\_\_，每一个特殊方法的方法名都是以双下划线开头和结尾。

#### deposit方法

```python
	def deposit(self, amount):
        if amount < Decimal('0.00'):
            raise ValueError('amount must be positive.')
        self.balance += amount
```

## 属性访问控制

一个类的**客户端代码**是使用类对象的任何代码。

大多数面向对象编程语言能够**封装**（或隐藏）一个对象的数据，被称为**私有数据**。

Python没有私有数据，但是通过命名约定将类的属性分为**直接访问属性**（attribute）和**间接访问属性**（property），间接访问属性命名以下划线开始，仅供类内使用，但这只是一种约定，其仍能在客户端代码中访问。

同样的，一些方法也可以作为仅在类内使用的**工具方法**，命名以单下划线开始。

对于直接访问属性而言，直接给数据属性赋值无法验证赋值的有效性，而间接访问属性不一样，将在下一节中具体讨论。

## Time类与property

### 试用Time类

```python
In [1]: from timewithproperties import Time

#创建一个Time对象
In [2]: wake_up = Time(hour = 6, minute = 30)
#Time类的__init__方法有hour、minute和second三个参数，每个参数的默认值为0

#显示Time对象
#__repr__特殊方法生成该对象的字符串表示
In [3]: wake_up
Out[3]: Time(hour=6, minute=30, second=0)

#__str__特殊方法创建12小时时钟格式的字符串
In [4]: print(wake_up)
6:30:00 AM

#通过property获取属性值
In [5]: wake_up.hour
Out[5]: 6
#property以方法的形式实现，此处调用hour方法返回数据属性的值而不是简单获取

#设置时间
In [6]: wake_up.set_time(hour = 7, minute = 45)

#通过property设置属性值
In [7]: wake_up.hour = 6
#property调用hour方法以6为参数，该方法会先验证值是否有效
```

### Time类的定义

#### \_\_init\_\_方法

```python
class Time:
    """Class Time with read-write properties."""
    def __init__(self, hour = 0, minute = 0, second = 0):
        self.hour = hour
        self.minute = minute
        self.second = second
```

#### 命名为hour的可读写property

以下代码定义了可公开访问、命名为hour的可读写property，用于操作名为_hour的数据属性，即间接访问。

property像是数据属性，但被实现为方法。

```python
	@property
    #每个property定义了一个getter方法用于获取一个数据属性的值，以@property装饰器开始
    def hour(self):
        return self._hour
    @hour.setter
    #选择性地定义一个setter方法用于设置一个数据属性的值，以@property_name.setter形式的装饰器开始
    def hour(self, hour):
        if not (0 <= hour <= 24):
            raise ValueError(f'Hour ({hour}) must be 0-23')
        self._hour = hour
    #单独使用wake_up.hour调用getter方法，用赋值语句wake_up.hour=8调用setter方法
    #可读写property既有getter方法也有setter方法，而只读property只有getter方法
```

#### 分别命名为minute和second的可读写property

```python
	@property
	def minute(self):
        return self.minute
    @minute.setter
    def minute(self, minute):
        if not (0 <= minute < 60):
            raise ValueError(f'Minute ({minute}) must be 0-59')
        self._minute = minute
    
    @property
    def second(self):
        return self._second
    @second.setter
    def second(self, second):
        if not (0 <= second < 60):
            raise ValueError(f'Second ({second}) must be 0-59')
        self._second = second
```

#### set_time方法

```python
	#使用set_time方法以单一的方法调用同时修改3个属性的值
    def set_time(self, hour = 0, minute = 0, second = 0):
        self.hour = hour
        self.minute = minute
        self.second = second
```

#### \_\_repr\_\_特殊方法

```python
	#在IPython会话中评估一个变量会隐式地执行内置函数repr，对应类的__repr__方法会被调用得到该对象的字符串表示
    def __repr__(self):
        return (f'Time(hour={self.hour}, minute={self.minute}, second={self.second})')
```

#### \_\_str\_\_特殊方法

```python
	#当使用内置函数str将一个对象转成字符串时（print函数隐式调用str），__str__方法被隐式调用
    def __str__(self):
        return (('12' if self.hour in (0, 12) else str(self.hour % 12)) + 
                f':{self.minute:0>2}:{self.second:0>2}' + 
                (' AM' if self.hour < 12 else ' PM'))
```

## 模拟“私有”属性

Python对象的属性始终是可访问的，但是对“私有”属性有命名约定。

如果要创建一个Time类的对象，并禁止下面的赋值语句：

$\texttt{wake}\_\texttt{up.}\_\texttt{hour = 100}$

则应该用以双下划线开始的“私有”属性名\_\_hour，因为Python会自动对“私有”属性加前缀\_ClassName（ClassName为属性所属的类名），如\_\_hour修饰为\_Time\_\_hour，这被称为**命名修饰**。如果使用\_\_hour对其赋值，会显示类中没有该属性。

## 类属性

一个类的每个对象都有关于类中数据属性的独立的值，这种属于对象的属性称为**实例属性**。

有些情况下，一个属性需要由一个类的所有对象共享。**类属性**用于表示类范围的信息，这个信息属于类，而不属于该类的一个特定对象。

例如在表示扑克牌的Card类中，我们会这样定义类属性：

```python
class Card:
    FACES = ['Ace', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'Jack', 'Queen', 'King']
    SUITS = ['Hearts', 'Diamonds', 'Clubs', 'Spades']
```

在类定义内部但在所有方法和property外，可以通过赋值定义一个类属性。每个Card对象不需要单独保存这两个列表就可以通过类名（Card.FACES和Card.SUITS）访问列表中的值。
___
**未完待续**
