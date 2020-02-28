### 1. 命令模式

![](https://github.com/Seizens/TyporaMarkdowmPic/blob/master/20200227_execute.png)

1. “命令”设计模式也可以通过把函数作为参数传递而简化
2. 各个命令可以有不同的接收者（实现操作的对象）。对PasteCommand来说，接收者是Document。对OpenCommand来说，接收者是应用程序
3. “命令”模式的目的是解耦调用操作的对象（调用者）和提供实现的对象（接收者）。在《设计模式：可复用面向对象软件的基础》所举的示例中，调用者是图形应用程序中的菜单项，而接收者是被编辑的文档或应用程序自身。
4. 这个模式的做法是，在二者之间放一个Command对象，让它实现只有一个方法（execute）的接口，调用接收者中的方法执行所需的操作。这样，调用者无需了解接收者的接口，而且不同的接收者可以适应不同的Command子类。调用者有一个具体的命令，通过调用execute方法执行。
5. 我们可以不为调用者提供一个Command实例，而是给它一个函数。此时，调用者不用调用command.execute（　），直接调用command（　）即可。MacroCommand可以实现成定义了__call__方法的类。这样，MacroCommand的实例就是可调用对象，各自维护着一个函数列表，供以后调用

```python
class MacroCommand:
    """一个执行一组命令的命令"""
    def __init__(self, commands):
        self.commands = list(commands)
        
    def __call__(self, *args, **kwargs):
        for command in self.commands:
            command()
```

使用一等函数对“命令”模式的重新审视到此结束。站在一定高度上看，这里采用的方式与“策略”模式所用的类似：把实现单方法接口的类的实例替换成可调用对象。毕竟，每个Python可调用对象都实现了单方法接口，这个方法就是__call__。



### 2. 装饰器

1. **函数装饰器用于在源码中“标记”函数，以某种方式增强函数的行为**

 假如有个名为decorate的装饰器:

```python
@decorate
def target():
	print("running target")
```

上述代码的效果与下述写法一样：

```python
def target()
	print("running target")
target = decorate(target)
```

两种写法的最终结果一样：上述两个代码片段执行完毕后得到的target不一定是原来那个target函数，而是decorate(target)返回的函数。即被装饰的函数会被替换。

```python
>>> def deco(func):
...     def inner():
...             print("running inner()")
...     return inner #1
... 
>>> @deco
... def target(): #2
...     print("running target()")
... 
>>> target()  #3
running inner()
>>> target  #4
<function deco.<locals>.inner at 0x7ffbac6f69d8>
```

❶ deco返回inner函数对象。

❷ 使用deco装饰target。

❸ 调用被装饰的target其实会运行inner。

❹ 审查对象，发现target现在是inner的引用。

严格来说，装饰器只是语法糖。如前所示，装饰器可以像常规的可调用对象那样调用，其参数是另一个函数。

装饰器的一大特性是，能把被装饰的函数替换成其他函数。第二个特性是，装饰器在加载模块时立即执行。

2. **装饰器加载模块是立即执行**

```python
registry = []

def register(func):
    print("running register(%s)" % func)
    registry.append(func)
    return func

@register
def f1():
    print("running f1()")

@register
def f2():
    print("running f2()")

def f3():
    print("running f3()")

def main():
    print('running main()')
    print('registry->', registry)
    f1()
    f2()
    f3()

if __name__ == '__main__':
    main()
```

直接执行的结果如下所示：

```bash
running register(<function f1 at 0x7f7d1e64a6a8>)
running register(<function f2 at 0x7f7d1e64a950>)
running main()
registry-> [<function f1 at 0x7f7d1e64a6a8>, <function f2 at 0x7f7d1e64a950>]
running f1()
running f2()
running f3()

Process finished with exit code 0
```

导入时

```python
>>> import registration
running register(<function f1 at 0x7ffbac6f6ae8>)
running register(<function f2 at 0x7ffbac6f6b70>)
>>> registration.registry
[<function f1 at 0x7ffbac6f6ae8>, <function f2 at 0x7ffbac6f6b70>]
```

(1) 装饰器通常在一个模块中定义，然后应用到其他模块中的函数上。

(2) 大多数装饰器会在内部定义一个函数，然后将其返回。

虽然上处register装饰器原封不动地返回被装饰的函数，但是这种技术并非没有用处。很多Python Web框架使用这样的装饰器把函数添加到某种中央注册处，例如把URL模式映射到生成HTTP响应的函数上的注册处。这种注册装饰器可能会也可能不会修改被装饰的函数。

### 3. 变量的作用域规则

```python
>>> def f1(a):
...     print(a)
...     print(b)
... 
>>> f1(3)
3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in f1
NameError: name 'b' is not defined
>>> b = 6 
>>> f1(3)
3
6
>>> b = 6
>>> def f2(a):
...     print(a)
...     print(b)
... 
>>> f2(2)
2
6
>>> def f3(a):
...     print(a)
...     print(b)
...     b = 9
... 
>>> f3(3)
3
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in f3
UnboundLocalError: local variable 'b' referenced before assignment
```

Python编译函数的定义体时，**它判断b是局部变量，因为在函数中给它赋值了。**生成的字节码证实了这种判断，Python会尝试从本地环境获取b。后面调用f2(3)时， f2的定义体会获取并打印局部变量a的值，但是尝试获取局部变量b的值时，发现b没有绑定值。

这不是缺陷，而是设计选择：Python不要求声明变量，但是假定在函数定义体中赋值的变量是局部变量。这比JavaScript的行为好多了，JavaScript也不要求声明变量，但是如果忘记把变量声明为局部变量（使用var），可能会在不知情的情况下获取全局变量。

如果在函数中赋值时想让解释器把b当成全局变量，要使用global声明

```python
>>> def f4(a):
...     global b
...     print(b)
...     print(a)
...     b = 9
... 
>>> f4(3)
6
3
>>> b
9
>>> f4(3)
9
3
>>> b = 40
>>> b
40
>>> f4(3)
40
3
```

### 4. 闭包

假设有个vag的函数，它的作用是计算不断增加的系列值的均值；

作用如下所示:

```python
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
```

**初学者可能如下使用类实现**

```python
>>> class Averager():
...     def __init__(self):
...             self.series = []
...     def __call__(self, new_value):
...             self.series.append(new_value)
...             total = sum(self.series)
...             return total/len(self.series)
... 
>>> avg = Averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
```

**使用高阶函数实现**

```python
>>> class Averager():
...     def __init__(self):
...             self.series = []
...     def __call__(self, new_value):
...             self.series.append(new_value)
...             total = sum(self.series)
...             return total/len(self.series)
... 
>>> avg = Averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
```

series是make_averager函数的局部变量，因为那个函数的定义体中初始化了series：series=[]。可是，调用avg(10)时，make_averager函数已经返回了，而它的本地作用域也一去不复返了。

在averager函数中，series是自由变量（free variable）。这是一个技术术语，指未在本地作用域中绑定的变量。下图所示：

![](https://github.com/Seizens/TyporaMarkdowmPic/blob/master/20200227_freevariable.png)

审查返回的averager对象，我们发现Python在__code__属性（表示编译后的函数定义体）中保存局部变量和自由变量的名称.

```python
>>> avg.__code__.co_varnames
('new_value', 'total')
>>> avg.__code__.co_freevars
('series',)
```

series的绑定在返回的avg函数的__closure__属性中。avg.__closure__中的各个元素对应于avg.__code__.co_freevars中的一个名称。这些元素是cell对象，有个cell_contents属性，保存着真正的值。

```python
>>> avg.__code__.co_freevars
('series',)
>>> avg.__closure__
(<cell at 0x7ffbb07ad2e8: list object at 0x7ffbac734c48>,)
>>> avg.__closure__[0].cell_contents
[10, 11, 12]
```

闭包是一种函数，它会保留定义函数时存在的自由变量的绑定，这样调用函数时，虽然定义作用域不可用了，但是仍能使用那些绑定。

注意，只有嵌套在其他函数中的函数才可能需要处理不在全局作用域中的外部变量。

### 5. nonlocal声明

```python
>>> def make_averager():
...     count = 0
...     total = 0
...     def averager(new_value):
...             count += 1
...             total += new_value
...             return total/count
...     return averager
... 
>>>
>>> avg = make_averager()
>>> avg(10)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 5, in averager
UnboundLocalError: local variable 'count' referenced before assignment
```

问题是，当count是数字或任何不可变类型时，count+=1语句的作用其实与count=count+1一样。因此，我们在averager的定义体中为count赋值了，这会把count变成局部变量。total变量也受这个问题影响。

但是对数字、字符串、元组等不可变类型来说，只能读取，不能更新。如果尝试重新绑定，例如count=count+1，其实会隐式创建局部变量count。这样，count就不是自由变量了，因此不会保存在闭包中。

nonlocal声明。它的作用是把变量标记为自由变量，即使在函数中为变量赋予新值了，也会变成自由变量。如果为nonlocal声明的变量赋予新值，闭包中保存的绑定会更新。

```python
>>> def make_averager():
...     count = 0
...     total = 0
...     def averager(new_value):
...             nonlocal count, total
...             count += 1
...             total += new_value
...             return total / count
...     return averager
... 
>>> 
>>> avg = make_averager()
>>> avg(10)
10.0
>>> avg(11)
10.5
>>> avg(12)
11.0
```

### 6. 继续装饰器

装饰器的典型行为：**把被装饰的函数替换成新函数，二者接受相同的参数，而且（通常）返回被装饰的函数本该返回的值，同时还会做些额外操作**。

**Python内置了三个用于装饰方法的函数：property、classmethod和staticmethod。**

**另一个常见的装饰器是functools.wraps，它的作用是协助构建行为良好的装饰器。**

**functools.lru_cache是非常实用的装饰器，它实现了备忘（memoization）功能。**

除了优化递归算法之外，lru_cache在从Web中获取信息的应用中也能发挥巨大作用。

```python
functools.lru_cache(maxsize=128, typed=False)
```

**maxsize参数指定存储多少个调用的结果。缓存满了之后，旧的结果会被扔掉，腾出空间。**为了得到最佳性能，maxsize应该设为2的幂。**typed参数如果设为True，把不同参数类型得到的结果分开保存**，即把通常认为相等的浮点数和整数参数（如1和1.0）区分开。顺便说一下，因为lru_cache使用字典存储结果，而且键根据调用时传入的定位参数和关键字参数创建，所以被lru_cache装饰的函数，它的所有参数都必须是可散列的。

单分派泛函数

```python
from functools import singledispatch
from collections import abc
import numbers
import html

@singledispatch #1
def htmlize(obj):
    content = html.escape(repr(obj))
    return '<pre>{}</pre>'.format(content)

@htmlize.register(str) #2
def _(text):            #3
    content = html.escape(text).replace('\n', '<br>\n')
    return '<p>{0}</p>'.format(content)

@htmlize.register(numbers.Integral) #4
def _(n):
    return '<pre> {0} (0x{0:x})'.format(n)

@htmlize.register(tuple) #5
@htmlize.register(abc.MutableSequence)
def _(seq):
    inner = '</li>\n<li>'.join(htmlize(item) for item in seq)
    return '<ul>\n<li>'+inner+'</li>\n</ul>'
```

❶ @singledispatch标记处理object类型的基函数。

❷ 各个专门函数使用@«base_function».register(«type»)装饰。

❸ 专门函数的名称无关紧要；_是个不错的选择，简单明了。

❹ 为每个需要特殊处理的类型注册一个函数。numbers.Integral是int的虚拟超类。

❺ 可以叠放多个register装饰器，让同一个函数支持不同类型。

注册的专门函数应该处理抽象基类（如numbers.Integral和abc.MutableSequence），不要处理具体实现（如int和list）。这样，代码支持的兼容类型更广泛。例如，Python扩展可以子类化numbers.Integral，使用固定的位数实现int类型。

使用抽象基类检查类型，可以让代码支持这些抽象基类现有和未来的具体子类或虚拟子类。

```python
>>> import htmlize
>>> htmlize.htmlize({1,2,3})
'<pre>{1, 2, 3}</pre>'
>>> htmlize.htmlize(abs)
'<pre>&lt;built-in function abs&gt;</pre>'
>>> htmlize.htmlize(42)
'<pre> 42 (0x2a)'
>>> print(htmlize.htmlize(['alpha', 66, {3, 2, 1}]))
<ul>
<li><p>alpha</p></li>
<li><pre> 66 (0x42)</li>
<li><pre>{1, 2, 3}</pre></li>
</ul>
```

因为Python不支持重载方法或函数，所以我们不能使用不同的签名定义htmlize的变体，也无法使用不同的方式处理不同的数据类型。在Python中，一种常见的做法是把htmlize变成一个分派函数，使用一串if/elif/elif，调用专门的函数，如htmlize_str、htmlize_int，等等。这样不便于模块的用户扩展，还显得笨拙：时间一长，分派函数htmlize会变得很大，而且它与各个专门函数之间的耦合也很紧密。

**functools.singledispatch**装饰器可以把整体方案拆分成多个模块，甚至可以为你无法修改的类提供专门的函数。**使用@singledispatch装饰的普通函数会变成泛函数（generic function）**：**根据第一个参数的类型，以不同方式执行相同操作的一组函数**。

singledispatch机制的一个显著特征是，你可以在系统的任何地方和任何模块中注册专门函数。如果后来在新的模块中定义了新的类型，可以轻松地添加一个新的专门函数来处理那个类型。此外，你还可以为不是自己编写的或者不能修改的类添加自定义函数。

@singledispatch不是为了把Java的那种方法重载带入Python。在一个类中为同一个方法定义多个重载变体，比在一个函数中使用一长串if/elif/elif/elif块要更好。但是这两种方案都有缺陷，因为它们让代码单元（类或函数）承担的职责太多。@singledispath的优点是支持模块化扩展：各个模块可以为它支持的各个类型注册一个专门函数。

**把@d1和@d2两个装饰器按顺序应用到f函数上，作用相当于f=d1(d2(f))。**

参数化装饰器基本上都涉及至少两层嵌套函数，如果想使用@functools.wraps生成装饰器，为高级技术提供更好的支持，嵌套层级可能还会更深