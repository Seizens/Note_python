#### **python之队列模块**

##### 1. 什么是队列

双端队列（deque，全名double-ended queue），是一种具有队列和栈的性质的数据结构。

双端队列中的元素可以从两端弹出，其限定插入和删除操作在表的两端进行。双端队列可以在队列任意一端入队和出队。



##### 2. 使用方式

```python
from collections import deque
```



##### 3. 方法说明

```python
class deque(object):
    """
    deque([iterable[, maxlen]]) --> deque object
    
    A list-like sequence optimized for data accesses near its endpoints.
    """
    def append(self, *args, **kwargs): # real signature unknown
        """ Add an element to the right side of the deque. """
        """ 添加一个元素，在队列右侧添加进去"""
        pass

    def appendleft(self, *args, **kwargs): # real signature unknown
        """ Add an element to the left side of the deque. """
        """ 添加一个元素，在队列左侧添加进去"""
        pass

    def clear(self, *args, **kwargs): # real signature unknown
        """ Remove all elements from the deque. """
        """清空队列所有内容"""
        pass

    def copy(self, *args, **kwargs): # real signature unknown
        """ Return a shallow copy of a deque. """
        """浅拷贝"""
        pass

    def count(self, value): # real signature unknown; restored from __doc__
        """ D.count(value) -> integer -- return number of occurrences of value """
        """如果时数值时，会计算总值"""
        return 0

    def extend(self, *args, **kwargs): # real signature unknown
        """ Extend the right side of the deque with elements from the iterable """
        """从右侧添加一堆元素进来"""
        pass

    def extendleft(self, *args, **kwargs): # real signature unknown
        """ Extend the left side of the deque with elements from the iterable """
        """从左侧添加一堆元素进来"""
        pass

    def index(self, value, start=None, stop=None): # real signature unknown; restored from __doc__
        """
        D.index(value, [start, [stop]]) -> integer -- return first index of value.
        Raises ValueError if the value is not present.
        """
        """查找一个元素第一次出现位置的索引地址"""
        return 0

    def insert(self, index, p_object): # real signature unknown; restored from __doc__
        """ D.insert(index, object) -- insert object before index """
        """在索引地址插入一个值"""
        pass

    def pop(self, *args, **kwargs): # real signature unknown
        """ Remove and return the rightmost element. """
        """从队列末尾开始删除元素"""
        pass

    def popleft(self, *args, **kwargs): # real signature unknown
        """ Remove and return the leftmost element. """
        """从队列头开始删除元素"""
        pass

    def remove(self, value): # real signature unknown; restored from __doc__
        """ D.remove(value) -- remove first occurrence of value. """
        """删除第一个查到的元素"""
        pass

    def reverse(self): # real signature unknown; restored from __doc__
        """ D.reverse() -- reverse *IN PLACE* """
        pass

    def rotate(self, *args, **kwargs): # real signature unknown
        """ Rotate the deque n steps to the right (default n=1).  If n is negative, rotates left. """
        """旋转列表，默认为旋转一部， 就像一个圆桌，每个人移动一次位置"""
        pass
```



##### 4. 举例说明

```python
>>> from collections import deque
>>> q = deque(maxlen=5)
>>> q.append(1)
>>> q.append(2)
>>> 
>>> q
deque([1, 2], maxlen=5)
>>> q.appendleft(-1)
>>> q
deque([-1, 1, 2], maxlen=5)
>>> q.appendleft(-2)
>>> q
deque([-2, -1, 1, 2], maxlen=5)
>>> q.pop()
2
>>> q
deque([-2, -1, 1], maxlen=5)
>>> q.popleft()
-2
>>> q
deque([-1, 1], maxlen=5)
>>> q.index(-1)
0
>>> q.extend((2,3,4))
>>> q
deque([-1, 1, 2, 3, 4], maxlen=5)
>>> q.extendleft((-3,-2))
>>> q
deque([-2, -3, -1, 1, 2], maxlen=5)
>>> q.reverse()
>>> q
deque([2, 1, -1, -3, -2], maxlen=5)
>>> q.remove(-3)
>>> q
deque([2, 1, -1, -2], maxlen=5)
>>> q.clear()
>>> q
deque([], maxlen=5)
>>> q
deque([1, 2, 3, 4], maxlen=5)
>>> q.rotate(2)  # 旋转2部
>>> q
deque([3, 4, 1, 2], maxlen=5)
```

 使用 `deque(maxlen=N)` 构造函数会新建一个固定大小的队列。当新的元素加入并且这个队列已满的时候， 最老的元素会自动被移除掉。

`deque` 类可以被用在任何你只需要一个简单队列数据结构的场合。 如果你不设置最大队列大小，那么就会得到一个无限大小队列，你可以在队列的两端执行添加和弹出元素的操作。

在队列两端插入或删除元素时间复杂度都是 `O(1)` ，区别于列表，在列表的开头插入或删除元素的时间复杂度为 `O(N)` 。