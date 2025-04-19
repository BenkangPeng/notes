### 第一章 Python数据模型

```python
import collections
from random import choice
Card = collections.namedtuple('Card' , ['rank' , 'suit'])
class FrenchDeck:
    ranks = [str(n) for n in range(2,11)] + list('JQKA')
    suits  = 'spades diamonds clubs hearts'.split()
    
    def __init__(self):
        self._cards = [Card(rank , suit) for rank in self.ranks 
                                         for suit in self.suits]
    def __len__(self):
        return len(self._cards)
    def __getitem__(self,position):
        return self._cards[position]
    
if __name__ == '__main__':
    deck = FrenchDeck()
    print(len(deck))
    print(deck[-1])
    print(deck[:3])
    for i in range(10):
        print(choice(deck))
```

* `namedtuple`

> In Python, NamedTuple is present inside the collections module. It provides a way to create simple, lightweight data structures similar to a class, but without the overhead of defining a full class. Like dictionaries, they contain keys that are hashed to a particular value. On the contrary, it supports both access from key-value and iteration, the functionality that dictionaries lack.

[Namedtuple in Python - GeeksforGeeks](https://www.geeksforgeeks.org/namedtuple-in-python/)

划重点:①一个轻量的数据结构类；②相比于字典，还有下标索引的功能。

##### **序列协议**

- 在 Python 中，序列是一种通用的数据结构，可以包含多个元素，并且支持**迭代**、**索引**和**切片**操作。
- 序列协议定义了一组**特殊方法**，例如 `__len__` 和 `__getitem__`，用于使对象表现得像序列一样。
- 如果一个类实现了这些方法，它的**实例**就可以像**列表**、**字符串**或**元组**一样进行操作。

`FrenchDeck`类中实现了`__len__`方法后，对实例化类`deck`执行`len(deck)`时，实际上调用的是`deck.__len__(self)` ; 同理，对`deck`执行切片、下标索引时实际调用的是`__getitem__`方法。

类似的，`C++`中可以通过重载`operator[]`来实现类似python的`__getitem__`  magic

**sorted**:

```python
for card in sorted(deck , key=card_value):
        print(card)
```

其中`card_value`不需要传入参数`card` , `sorted`会自动将`deck`中每一个`card`传入`key`中,不需要显式传参。



```python
from math import sqrt
class Vector:
    def __init__(self , x=0 , y=0):
        self.x = x
        self.y = y
    def __repr__(self) -> str:
        return 'Vector(%r,%r)'%(self.x , self.y)
    def __abs__(self):
        return sqrt(self.x**2 + self.y**2)
    def __bool__(self):
        return bool(self.x or self.y)
    def __add__(self , other):
        return Vector(self.x+other.x , self.y+other.y)
    def __mul__(self , scalar):
        return Vector(self.x*scalar , self.y*scalar) 
if __name__ == '__main__':
    v1 = Vector(3,4)
    v2 = Vector(2,2)
    v3 = Vector(0,0)
    print(v1+v2)
    print(v1*2)
    print(abs(v1))
    print(bool(v3))
```

### 第二章 数据结构

##### **列表推导与生成器表达式**

```python

if __name__ == '__main__':
    colors = ['black', 'white']
    sizes = ['S', 'M', 'L']
    tshirt = [(color,size) for color in colors
                            for size in sizes]
    print(tshirt)
    symbols = '$¢£¥€¤'
    a = list(ord(symbol) for symbol in symbols if ord(symbol) > 127)
    b = [ ord(symbol) for symbol in symbols if ord(symbol) > 127 ]
    print(a , b)
```

> 虽然也可以用列表推导来初始化元组、数组或其他序列类型，但是生成
> 器表达式是更好的选择。这是因为生成器表达式背后遵守了迭代器协
> 议，可以逐个地产出元素，而不是先建立一个完整的列表，然后再把这
> 个列表传递到某个构造函数里。前面那种方式显然能够节省内存。

##### **元组**

```python
if __name__ == '__main__':
    lax_coordinates = (33.9425, -118.408056)
    city, year, pop, chg, area = ('Tokyo', 2003, 32450, 0.66, 8014)
    traveler_ids = [('USA', '31195855'), ('BRA', 'CE342567'),('ESP', 'XDA205856')]
    for passport in sorted(traveler_ids):
        print('%s/%s' % passport)
```

> 我们把元组 ('Tokyo', 2003, 32450, 0.66, 8014) 里
> 的元素分别赋值给变量 city、year、pop、chg 和 area，而这所有的
> 赋值我们只用一行声明就写完了。同样，在后面一行中，一个 % 运算符
> 就把 passport 元组里的元素对应到了 print 函数的格式字符串空档
> 中。这两个都是对元组拆包的应用。

不使用中间变量交换两个变量的值

```python
a , b = b , a
```

实际此处涉及到元组的解包:python首先解析右值，组成元组(b,a),再解包赋值给a,b

##### ***运算符**

```python
def devmode(a , b):
    return (a//b , a%b)
t = (12,7)
print(devmode(*t))
a , b , *_ = range(5)
print(a,b,_)
```

##### **嵌套元组**

```python
if __name__ == '__main__':
    metro_areas = [
    ('Tokyo','JP',36.933,(35.689722,139.691667)), 
    ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889)),
    ('Mexico City', 'MX', 20.142, (19.433333, -99.133333)),
    ('New York-Newark', 'US', 20.104, (40.808611, -74.020386)),
    ('Sao Paulo', 'BR', 19.649, (-23.547778, -46.635833)),
    ]
    print('{:15} | {:^9} | {:^9}'.format('', 'lat.', 'long.'))
    fmt = '{:15} | {:9.4f} | {:9.4f}'
    for name, cc, pop, (latitude, longitude) in metro_areas:
        if longitude <= 0:
            print(fmt.format(name, latitude, longitude))
```

`{:15}`表示占15格 ， `{:^9}`表示占9格，字符串居中 ， `{:>9}`右对齐占9格，`{:9.4f}`占9格，保留四位小数的浮点数



##### **具名元组collections.namedtuple**

```python
from collections import namedtuple
if __name__ == '__main__':
    city = namedtuple('city' , ['name' , 'country' , 'pop' , 'coordinates'])
    tokyo = city('tokyo' , 'Japan' , '36.933' , (35,139))
    print(tokyo.pop , tokyo.coordinates)

    print(city._fields)
    delhi_data = ('Delhi NCR', 'IN', 21.935, (28.613889, 77.208889))
    delhi = city._make(delhi_data)
    print(delhi._asdict)
```

`._fields`返回具名元素所有的字段名称

`._make`支持以**元组(或列表)**创建具名元组

`._asdict`把具名元组以 collections.OrderedDict 的形式返回，我们可以利用它来把元组里的信息友好地呈现出来。



##### **切片**

```python
s = 'bicycle'
print(s[::3] , s[::-1] , s[::-2])
print(s[1:5:2] , s[5:1:-2])
```

`start:stop:step`    `step`为负值表示反向切片

##### **对序列使用***

```python
if __name__ == '__main__':
    l = [1,2,3]
    print(l * 3 , 3 * 'abc')
    board = [['_']*3 for i in range(3)]
    print(board)
    board[1][2] = 'X'
    print(board)
    board_ = [['_']*3]*3
    board_[1][2] = 'X'
    print(board_)

```

```shell
$ python chapter2.py 
[1, 2, 3, 1, 2, 3, 1, 2, 3] abcabcabc
[['_', '_', '_'], ['_', '_', '_'], ['_', '_', '_']]
[['_', '_', '_'], ['_', '_', 'X'], ['_', '_', '_']]
[['_', '_', 'X'], ['_', '_', 'X'], ['_', '_', 'X']]
```

`board_ = [['_']*3]*3`  这种方法创建了一个包含三个引用的列表，修改其中一个列表，其他两个列表都会变化。



### 第三章 字典和集合

##### **字典初始化**

```python
if __name__ == '__main__':
    a = dict(one=1, two=2, three=3)
    b = {'one': 1, 'two': 2, 'three': 3}
    c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
    d = dict([('two', 2), ('one', 1), ('three', 3)])
    e = dict({'three': 3, 'one': 1, 'two': 2})
    print(a == b == c == d == e)#以上五种字典初始化方式等价
```

##### **字典推导**

```python
DIAL_CODES = [(86, 'China'),(91, 'India'),(1, 'United States'),(62, 'Indonesia'),(55, 'Brazil'),(92, 'Pakistan'),(880, 'Bangladesh'),(234, 'Nigeria'),(7, 'Russia'),(81, 'Japan')]
country_code = {country : code for code,country in DIAL_CODES if code > 66}
print(country_code)
```

##### **集合**

```python
if __name__ == '__main__':
    l = [1,2,3,4,5,6,7,1,2,3]
    s = set(l) 
    print(s)    

    s2 = {4,6,9}
    print(s&s2)
    print(s|s2)
    print(s-s2)

    s3 = set()
    s3.add(1)
    # s3.pop()
    print(s3)
```

`s = set(l)`会将列表`l`中元素去重

`s&s2`求交集，`s|s2`求并集，`s-s2`求补集

### 第五章 一等函数

##### **把函数当作对象**

```python

def factorial(n : int):
    '''return n!'''
    return 1 if n < 2 else n * factorial(n-1)
if __name__ == '__main__':
    print(factorial(10))
    print(type(factorial))
    print(factorial.__doc__)
    fact = factorial
    print(list(map(fact , range(11))))
```

`type(factorial)`会显示`<class 'function'>` , 表明`factorial`是一个`function`类

`factorial.__doc__`返回函数的说明文档`return n!`

`fact = factorial`将`factorial`类实例化了一份给`fact`

可以将`fact`作为参数传给`高阶函数`

`map `函数返回一个可迭代对象，里面的元素是把第一个参数(一个函数)应用到第二个参数（一个可迭代对象，这里是`range(11)`）中各个元素上得到的结果。

##### **高阶函数**

接受函数为参数，或者把函数作为结果返回的函数是高阶函数higherorder function

例如`sorted` , `map`

```python
if __name__ == '__main__':
    fruits = ['strawberry', 'fig', 'apple', 'cherry', 'raspberry', 'banana']
    print(sorted(fruits , key=len))#按长度排序
    print(list(map(str.upper , fruits)))
    print( [word.upper() for word in fruits ] )
```

##### **可调用类**

```python
import random
class BingoCage:
    def __init__(self , item):
        self.item = list(item)
        random.shuffle(self.item)
    def pick(self):
        try:
            return self.item.pop()
        except IndexError:
            raise LookupError('pick from empty BingoCage')
    def __call__(self):
        return self.pick()
if __name__ == '__main__':
    print(type(abs))
    print(type(str))
    print(type(13))
    print([callable(obj) for obj in (abs , str , 13)])

    bingo = BingoCage(range(20))
    print(bingo.item)
    print(bingo())
    print(callable(bingo))
```

```shell
$ python chapter5.py 
<class 'builtin_function_or_method'>
<class 'type'>
<class 'int'>
[True, True, False]
[5, 4, 2, 19, 10, 7, 3, 9, 1, 13, 16, 12, 0, 18, 11, 6, 15, 14, 8, 17]
17
True
```

`abs` , `str`均可调用(`callable`)   `13`不可调用

如果要使某个对象可调用，需要增加`__call__`方法



### 第六章 使用一等函数实现设计模式

##### **重构“策略”模式(ABC)**

```python
from abc import ABC , abstractmethod
from collections import namedtuple

Customer = namedtuple('Customer' , ['name' , 'fidelity']) #fidelity:忠诚度，保真度

class LineItem:
    def __init__(self , product , quantity , price):
        self.product = product
        self.quantity = quantity
        self.price = price

    def total(self):
        return self.quantity * self.price


class Order:#order:订单
    def __init__(self,customer,cart,promotion=None):#promotion:促销
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion

    def total(self):
        if not hasattr(self,'__total'):
            self.__total = sum(item.total() for item in self.cart)
        return self.__total
    
    def due(self):#due:应收
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion.discount(self)
        return self.total() - discount
    
    def __repr__(self) -> str:
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(),self.due())
    
class Promotion(ABC):

    @abstractmethod
    def discount(self,order):
        '''返回折扣金额'''
        pass

class FidelityPromo(Promotion):# 第一个具体策略
    '''为积分为1000或以上的顾客提供5%折扣'''
    def discount(self, order):
        return order.total() * 0.05 if order.customer.fidelity >= 1000 else 0
    
class BulkItemPromo(Promotion):# 第一个具体策略
    '''单个商品为20个或以上时提供10%折扣'''
    
    def discount(self, order):
         discount = 0
         for item in order.cart:
            if item in order.cart:
                if item.quantity >= 20:
                    discount += item.total() * 0.1
         return discount

class LargeOrderPromo(Promotion):# 第三个具体策略
    '''订单中的不同商品达到10个或以上时提供7%折扣'''
    def discount(self, order):
        distinct_items = {item.product for item in order.cart}
        if len(distinct_items) >= 10:
            return order.total * 0.07
        return 0

if __name__ == '__main__':
    joe = Customer('John Doe', 0)
    ann = Customer('Ann Smith', 1100)
    cart = [LineItem('banana', 4, .5), LineItem('apple', 10, 1.5),LineItem('watermellon', 5, 5.0)]

    print(Order(joe , cart , FidelityPromo()))
    print(Order(joe , cart , BulkItemPromo()))
    print(Order(ann , cart , FidelityPromo()))
```

* 三种不同的营销策略是写成三个类的`FidelityPromo` `BulkItemPromo` `FidelityPromo` , 这三个类都是继承`Promotion` ABC基类的。ABC基类的用处，我目前感觉说就是提供了一个模板，子类继承模板时就必须完善模板中的方法。例如，`Promotion`基类里有`abstractmethod`的`discount`方法，那么三个策略类继承`Promotion`时就必须严格按照`Promotion`的`discount`方法格式重新去定义。

好吧，似乎`newbing`的解释更严谨:

在Python编程中，抽象基类（Abstract Base Classes，简称ABC）是一种用于**定义接口和规范**的强大工具。它们提供了一种方式来**确保类遵循特定的约定**，从而增强了代码的可读性、可维护性和可扩展性[link1](https://docs.python.org/zh-cn/3/library/abc.html)    [link2](https://zhuanlan.zhihu.com/p/682337642)。

以下是使用ABC的主要原因：

1. **强制接口规范**：抽象基类可用于**强制**类遵循特定的接口规范。例如，假设您正在开发一个图形库，您可以创建一个 Drawable 抽象基类，**要求所有图形类都实现** draw () 方法。这样可以确保所有图形类都有相同的绘制方法，提高了代码的可维护性和一致性。
2. **类型检查和文档**：使用抽象基类可以改进代码的类型检查和文档。例如，可以创建一个 Database 抽象基类，要求所有数据库连接类都实现 connect () 和 execute () 方法。这样，可以在文档中明确说明数据库连接类的接口，并确保正确的类型检查。
3. **设计模式实现**：抽象基类在代码的类型检查、文档编写和设计模式实现中发挥着重要作用。例如，可以使用抽象基类实现策略模式，其中不同的算法被封装成策略类，并由上下文类选择并执行。



#####  @property  @staticmethod

```python
class Math:
    def __init__(self,radius):
        self.r = radius
    
    @property
    def area(self):
        return 3.14 * (self.r ** 2)
    
    @property
    def perimeter(self):
        return 2 * 3.14 * self.r
    
    @staticmethod
    def show_():
        print('hello')
if __name__ == '__main__':
    circle = Math(10)
    print(circle.area)
    # print(Math.perimeter(circle))
    Math.show_()
```

`@property`就是将方法修饰成类的属性，调用时可以直接`circle.area`而不是`circle.area()`

`@staticmethod`将方法定义成静态方法，即使不用实例化类，也能通过`<class_name>.<staticmethod>()`调用

其实上面购物的例子中用基类太鸡肋了。python中函数就是对象，可以将三个营销策略改写成函数:

```python
from collections import namedtuple

Customer = namedtuple('Customer' , ['name' , 'fidelity']) #fidelity:忠诚度，保真度

class LineItem:
    def __init__(self , product , quantity , price):
        self.product = product
        self.quantity = quantity
        self.price = price

    def total(self):
        return self.quantity * self.price
class Order:#order:订单
    def __init__(self,customer,cart,promotion=None):#promotion:促销
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion

    def total(self):
        if not hasattr(self,'__total'):
            self.__total = sum(item.total() for item in self.cart)
        return self.__total
    
    def due(self):#due:应收
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion(self)
        return self.total() - discount
    
    
    def __repr__(self) -> str:
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(),self.due())
    

def FidelityPromo(order):# 第一个具体策略
    '''为积分为1000或以上的顾客提供5%折扣'''
    return order.total() * 0.05 if order.customer.fidelity >= 1000 else 0
    
def BulkItemPromo(order):# 第一个具体策略
    '''单个商品为20个或以上时提供10%折扣'''
    
    discount = 0
    for item in order.cart:
        if item in order.cart:
            if item.quantity >= 20:
                discount += item.total() * 0.1
    return discount

def LargeOrderPromo(order):# 第三个具体策略
    '''订单中的不同商品达到10个或以上时提供7%折扣'''
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total * 0.07
    return 0

def best_promo(order):#求出最佳策略
    promotion = [FidelityPromo , BulkItemPromo , FidelityPromo]
    return max(promo(order) for promo in promotion)


if __name__ == '__main__':
    joe = Customer('John Doe', 0)
    ann = Customer('Ann Smith', 1100)
    cart = [LineItem('banana', 4, .5), LineItem('apple', 10, 1.5),LineItem('watermellon', 5, 5.0)]
    banana_cart = [LineItem('banana', 30, .5),LineItem('apple', 10, 1.5)]

    print(Order(joe , cart , FidelityPromo))
    print(Order(ann , banana_cart , BulkItemPromo) )

    print(best_promo(Order(joe,cart)))
    print(best_promo(Order(ann, banana_cart)))
```

