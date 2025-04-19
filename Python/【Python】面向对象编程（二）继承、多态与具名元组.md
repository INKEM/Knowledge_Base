> 参考书籍：《Python程序设计 人工智能案例实践》[美] 保罗·戴特尔 哈维·戴特尔 著
>
> 码字不易，求点赞收藏加关注
>
>有问题欢迎评论区讨论

@[TOC](目录)
# Python面向对象编程（二）
## 继承：基类和子类

一个类的一个对象经常也是另一个类的一个对象，其中一个类是基类，另一个类是子类。基类趋向于表示更一般的事物，而子类趋向于表示更具体的事物，每个子类对象都是其基类的一个对象，一个基类可以有很多子类。以下是一些例子：

| 基类        | 子类                                       |
| ----------- | ------------------------------------------ |
| Student     | GraduateStudent, UndergraduateStudent      |
| Shape       | Circle, Triangle, Rectangle, Sphere, Cube  |
| Loan        | CarLoan, HomeImprovementLoan, MortgageLoan |
| Employee    | Faculty, Staff                             |
| BankAccount | CheckingAccount, SavingsAccount            |

继承关系形成了树形层次结构，基类与它的子类是一种层次关系。下图展示了一个简单的类层次结构，也被称为**继承层次结构**。对于单继承，一个类从一个基类派生；对于多重继承，一个子类对两个或更多基类进行继承。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6ccfa077904c4a46980c3622f6562491.png#pic_center)


## 构建继承层次结构：引入多态性

在本例，我们给出基类CommissionEmplyee，然后通过继承CommissionEmployee类创建子类SalariedCommissionEmployee。

### 基类CommissionEmployee（佣金员工）

```python
from decimal import Decimal

class CommissionEmployee:
    """An employee who gets paid commission based on gross sales."""
    
    #创建数据属性，含义分别是名、姓、社保号、总销售额、佣金率
    def __init__(self, first_name, last_name, ssn, gross_sales, commission_rate):
        self._first_name = first_name
        self._last_name = last_name
        self._ssn = ssn
        self.gross_sales = gross_sales
        self.commission_rate = commission_rate
        
    #first_name, last_name, ssn三个只读property
    @property
    def first_name(self):
        return self._first_name
    
    @property
    def last_name(self):
        return self._last_name
    
    @property
    def ssn(self):
        return self._ssn
    
    #gross_sales, commission_rate两个读写property
    @property
    def gross_sales(self):
        return self._gross_sales
    
    @gross_sales.setter
    def gross_sales(self, sales):
        if sales < Decimal('0.00'):
            raise ValueError('Gross sales must be >= to 0')
        self._gross_sales = sales
    
    @property
    def commission_rate(self):
        return self._commission_rate
    
    @commission_rate.setter
    def commission_rate(self, rate):
        if not (Decimal('0.0') < rate < Decimal('1.0')):
            raise ValueError('Interest rate must be greater than 0 and less than 1')
        self._commission_rate = rate
    
    #earnings方法：计算并返回一个CommissionEmployee对象的收入
    def earnings(self):
        return self.gross_sales * self.commission_rate
    
    #返回字符串表示
    def __repr__(self):
        return ('CommissionEmployee: ' +
                f'{self.first_name} {self.last_name}\n' +
                f'social security number: {self.ssn}\n' +
                f'gross sales: {self.gross_sales:.2f}\n' +
                f'commission rate: {self.commission_rate:.2f}')
```

### 子类SalariedCommissionEmployee（带薪佣金员工）

```python
from commissionemployee import CommissionEmployee
from decimal import Decimal

#为了从一个类继承，必须首先在括号内导入该类的定义
class SalariedCommissionEmployee(CommissionEmployee):
    """An employee who gets paid a salary plus commission based on gross sales."""
    
    #该子类在基类的基础上新增了底薪base_salary属性
    def __init__(self, first_name, last_name, ssn, gross_sales, commission_rate, base_salary):
        #通过内置函数super找到并调用基类的__init__方法用以初始化继承过来的数据属性
        super().__init__(first_name, last_name, ssn, gross_sales, commission_rate)
        self.base_salary = base_salary
    
    @property
    def base_salary(self):
        return self._base_salary
    
    @base_salary.setter
    def base_salary(self, salary):
        if salary < Decimal('0.00'):
            raise ValueError('Base salary must be >= to 0')
        self._base_salary = salary
    
    #重写earnings方法
    def earnings(self):
        #通过内置函数super调用基类的earnings方法得到销售提成收入，与底薪相加计算出总收入
        return super().earnings() + self.base_salary
    
    #重写__repr__方法
    def __repr__(self):
        #通过内置函数super调用基类的__repr__方法并将其返回的字符串与底薪信息拼接
        return ('Salaried' + super().__repr__() +
                f'\nbase salary: {self.base_salary.2f}')
```

### 以多态方式处理CommissionEmployee和SalariedCommissionEmployee

因为通过继承，子类的每个对象也可以被视为该子类的基类的对象，所以可以将基类对象和基类的子类对象都视为基类对象放在一起遍历，用基类的方法进行一般化处理。以下例子将一个CommissionEmployee对象和一个SalariedCommissionEmployee对象放在一个列表里遍历输出。

```python
In [1]: from commissionemployee import CommissionEmployee

In [2]: from salariedcommissionemployee import SalariedCommissionEmployee

In [3]: from decimal import Decimal

In [4]: c = CommissionEmployee('Sue', 'Jones', '333-33-3333',
   ...:     Decimal('10000.00'), Decimal('0.06'))

In [5]: s = SalariedCommissionEmployee('Bob', 'Lewis', '444-44-4444',
   ...:     Decimal('5000.00'), Decimal('0.04'), Decimal('300.00'))

In [6]: employees = [c, s]

In [7]: for employee in employees:
   ...:     print(employee)
   ...:     print(f'{employee.earnings():,.2f}\n')
CommissionEmployee: Sue Jones
social security number: 333-33-3333
gross sales: 20000.00
commission rate: 0.10
2,000.00

SalariedCommissionEmployee: Bob Lewis
social security number: 444-44-4444
gross sales: 10000.00
commission rate: 0.05
base salary: 1000.00
1,500.00
```

## 鸭子类型和多态性

其他大多数面向对象编程语言，需要通过继承来实现多态。Python更加灵活，借助被称为**鸭子类型**的概念来实现多态。只要不同类的定义包含相同的方法，那么它们的对象就可以放在一起用该相同的方法遍历。

> 如果它看起来像鸭子，像鸭子一样嘎嘎叫，那么它就是鸭子。

```python
#创建一个没有继承CommissionEmployee类但同样拥有__repr__方法和earnings方法的类
In [8]: class WellPaidDuck:
   ...:     def __repr__(self):
   ...:         return 'I am a well-paid duck'
   ...:     def earnings(self):
   ...:         return Decimal('1_000_000.00')

In [9]: d = WellPaidDuck()

In [10]: employees = [c, s, d]

In [10]: for employee in employees:
    ...:     print(employee)
    ...:     print(f'{employee.earnings():,.2f}\n')
CommissionEmployee: Sue Jones
social security number: 333-33-3333
gross sales: 20000.00
commission rate: 0.10
2,000.00

SalariedCommissionEmployee: Bob Lewis
social security number: 444-44-4444
gross sales: 10000.00
commission rate: 0.05
base salary: 1000.00
1,500.00

I am a well-paid duck
1,000,000.00
```

## 具名元组

我们已经使用元组将多个数据属性聚合到一个对象中。Python标准库的collections模块还提供了具名元组，可以按名字而不是索引号引用元组的成员。

```python
In [1]: from collections import namedtuple

#namedtuple函数创建了内置元组类型的一个子类类型
#第一个参数是新创建的类型的名字，第二个参数是一个字符串列表，用于表示引用新类型成员的标识符
In [2]: Card = namedtuple('Card', ['face', 'suit'])

#创建一个Card对象
In [3]: card = Card(face = 'Ace', suit = 'Spades')

#通过名字访问其成员
In [4]: card.face
Out[4]: 'Ace'

In [5]: card.suit
Out[5]: 'Spades'
```

___

**未完待续**

