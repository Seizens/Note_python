### 1. 经典的”策略“模式

![](https://github.com/Seizens/TyporaMarkdowmPic/blob/master/20200227_jindiancelue.png)

《设计模式：可复用面向对象软件的基础》一书是这样概述“策略”模式的：定义一系列算法，把它们一一封装起来，并且使它们可以相互替换。本模式使得算法可以独立于使用它的客户而变化。

电商领域有个功能明显可以使用“策略”模式，即根据客户的属性或订单中的商品计算折扣。

假如一个网店制定了下述折扣规则。

有1000或以上积分的顾客，每个订单享5%折扣。

同一订单中，单个商品的数量达到20个或以上，享10%折扣。

订单中的不同商品达到10个或以上，享7%折扣。

简单起见，我们假定一个订单一次只能享用一个折扣。

“策略”模式的UML类图，其中涉及下列内容。

**上下文**

把一些计算委托给实现不同算法的可互换组件，它提供服务。在这个电商示例中，上下文是Order，它会根据不同的算法计算促销折扣。

**策略**

实现不同算法的组件共同的接口。在这个示例中，名为Promotion的抽象类扮演这个角色。

**具体策略**

“策略”的具体子类。fidelityPromo、BulkPromo和LargeOrderPromo是这里实现的三个具体策略。

代码实现如下所是。

```python
from abc import ABC, abstractmethod
from collections import namedtuple

#客户信息
Customer = namedtuple('Customer', 'name fidelity')

#商品信息
class LineItem:
    def __init__(self, product, quantity, price):
        self.product = product   #商品名
        self.quantity = quantity #数量
        self.price = price       #价格

    def total(self):
        return self.price * self.quantity

#订单信息   上下文
class Order:
    def __init__(self, customer, cart, promotion=None):
        self.customer = customer  #顾客信息
        self.cart = list(cart)	  #商品列表
        self.promotion = promotion  #打折促销方式

    #计算总价，不包含打折之后
    def total(self): 
        if not hasattr(self, '__total'):
            self.__total = sum(item.total() for item in self.cart)
        return self.__total
	
    #实际支付
    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion.discount(self)
        return self.total() - discount

    def __repr__(self):
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(), self.due())

# 打折基类  #策略：抽象基类
class Promotion(ABC):
    @abstractmethod
    def discount(self, order):
        """返回折扣金额(正值)"""

# 第一个具体策略  
class FidelityPromo(Promotion):
    """为积分大于1000或者以上的顾客提供5%的折扣"""
    def discount(self, order):
        return order.total() * .05 if order.customer.fidelity >1000 else 0

# 第二个具体策略
class BulkItemPromo(Promotion):
    """单个商品为20或以上的顾客提供10%的折扣"""
    def discount(self, order):
        discount = 0
        for item in order.cart:
            if item.quantity >= 20:
                discount += item.total() * .1
        return discount

# 第三个具体策略
class LargeOrderPromo(Promotion):
    """订单中不同商品的总类达到10或者10个以上的提供7%的折扣"""
    def discount(self, order):
        distinct_items = {item.product for item in order.cart}
        if len(distinct_items) >= 10:
            return order.total() * .07
        return 0
```

使用如下所示：

```python
>>> from orderpromo import *
>>> joe = Customer('Joho Doe', 0)
>>> ann = Customer('Ann Smith', 1100)
>>> cart = [LineItem('banana', 4, 0.5),
...     LineItem('apple', 10, 1.5),
...     LineItem('watermellon', 5, 5.0)]
>>> Order(joe, cart, FidelityPromo())
<Order total: 42.00 due: 42.00>
>>> Order(ann, cart, FidelityPromo())
<Order total: 42.00 due: 39.90>
>>> banana_cart = [LineItem('banana', 30, 0.5)]
>>> O
OSError(        Order(          OverflowError(  
>>> Order(joe, banana_cart, BulkItemPromo())
<Order total: 15.00 due: 13.50>
>>> banana_cart = [LineItem('banana', 30, 0.5),
...     LineItem('apple', 10, 1.5)]
>>> Order(joe, banana_cart, BulkItemPromo())
<Order total: 30.00 due: 28.50>
>>> long_order=[LineItem(str(item_code),1 , 1.0) for item_code in range(10)]
>>> Order(joe, long_order, LargeOrderPromo())
<Order total: 10.00 due: 9.30>
>>> Order(joe, ca)
callable(  cart       
>>> Order(joe, ca)
callable(  cart       
>>> Order(joe, cart, LargeOrderPromo())
<Order total: 42.00 due: 42.00>
```



### 2. 使用函数式实现策略

```python
from collections import namedtuple

Customer = namedtuple('Customer', 'name fidelity')

class LineItem:
    def __init__(self, product, quantity, price):
        self.product = product
        self.quantity = quantity
        self.price = price

    def total(self):
        return self.price * self.quantity

class Order:
    def __init__(self, customer, cart, promotion=None):
        self.customer = customer
        self.cart = list(cart)
        self.promotion = promotion

    def total(self):
        if not hasattr(self, '__total'):
            self.__total = sum(item.total() for item in self.cart)
        return self.__total

    def due(self):
        if self.promotion is None:
            discount = 0
        else:
            discount = self.promotion.discount(self)
        return self.total() - discount

    def __repr__(self):
        fmt = '<Order total: {:.2f} due: {:.2f}>'
        return fmt.format(self.total(), self.due())

def fidelity_promo(order):
    return order.total() * .05 if order.customer.fidelity >1000 else 0

def bulk_item_promo(order):
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount

def large_order_promo(order):
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total() * .07
    return 0
```

使用如下所示：

```python
>>> from orderfunc import *
>>> joe = Customer('Joho Doe', 0)
>>> ann = Customer('Ann Smith', 1100)
>>> cart = [LineItem('banana', 4, 0.5),
...     LineItem('apple', 10, 1.5),
...     LineItem('watermellon', 5, 5.0)]
>>> Order(joe, cart, fidelity_promo)
<Order total: 42.00 due: 42.00>
>>> Order(ann, cart, fidelity_promo)
<Order total: 42.00 due: 39.90>
```

### 3. 选择最佳的策略模式

```python
promos = [fidelity_promo, bulk_item_promo, large_order_promo]
def best_promo(order):
    return max(promo(order) for promo in promos)
```

但是如果新加新的折扣，则无法享受,修改如下

```python
promos = [globals()[name] for name in globals() if name.endswith('_promo') and name != 'best_promo']
def best_promo(order):
    return max(promo(order) for promo in promos)
```

### 4. 使用装饰器改良策略模式

```python
promos = []
def promotion(func):
    promos.append(func)
    return func

@promotion
def fidelity_promo(order):
    return order.total() * .05 if order.customer.fidelity >1000 else 0

@promotion
def bulk_item_promo(order):
    discount = 0
    for item in order.cart:
        if item.quantity >= 20:
            discount += item.total() * .1
    return discount

@promotion
def large_order_promo(order):
    distinct_items = {item.product for item in order.cart}
    if len(distinct_items) >= 10:
        return order.total() * .07
    return 0

def best_promo(order):
    return max(promo(order) for promo in promos)
```

(1) 促销策略函数无需使用特殊的名称（即不用以_promo结尾）。

(2) @promotion装饰器突出了被装饰的函数的作用，还便于临时禁用某个促销策略：只需把装饰器注释掉。

(3) 促销折扣策略可以在其他模块中定义，在系统中的任何地方都行，只要使用@promotion装饰即可。