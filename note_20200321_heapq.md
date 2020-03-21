#### Python模块之heapq

##### 1. 什么是堆

堆是一种数据结构，一种叫做完全二叉树的数据结构。

大顶堆：每个节点的值都大于或者等于它的左右子节点的值。

小顶堆：每个节点的值都小于或者等于它的左右子节点的值。

![](https://github.com/Seizens/TyporaMarkdowmPic/blob/master/20200321_heapq.png)

如上所示，就是两种堆。

如果我们把这种逻辑结构映射到数组中，就是下边这样

| 9    | 5    | 8    | 2    | 3    | 4    | 7    | 1    |

| 1    | 3    | 5    | 4    | 2    | 8    | 9    | 7    |

从这里我们可以得出以下性质（重点）

对于大顶堆：arr[i] >= arr[2i + 1] && arr[i] >= arr[2i + 2]

对于小顶堆：arr[i] <= arr[2i + 1] && arr[i] <= arr[2i + 2]



##### 2. 使用方式

```python
import heapq
```



##### 3. 方法说明

```python
__all__ = ['heappush', 'heappop', 'heapify', 'heapreplace', 'merge',
           'nlargest', 'nsmallest', 'heappushpop']

def heappush(heap, item):
    """Push item onto heap, maintaining the heap invariant."""
    """往堆中添加元素，仍然保持堆的性质"""

def heappop(heap):
    """Pop the smallest item off the heap, maintaining the heap invariant."""
    """从堆中去掉最小的一个元素"""

def heapreplace(heap, item):
    """Pop and return the current smallest value, and add the new item.

    This is more efficient than heappop() followed by heappush(), and can be
    more appropriate when using a fixed-size heap.  Note that the value
    returned may be larger than item!  That constrains reasonable uses of
    this routine unless written as part of a conditional replacement:

        if item > heap[0]:
            item = heapreplace(heap, item)
    """
    """删除最小的元素，然后添加一个新的元素"""

def heappushpop(heap, item):
    """Fast version of a heappush followed by a heappop."""

def heapify(x):
    """Transform list into a heap, in-place, in O(len(x)) time."""
    """将一个list转化为一个小堆"""
    n = len(x)
    # Transform bottom-up.  The largest index there's any point to looking at
    # is the largest with a child index in-range, so must have 2*i + 1 < n,
    # or i < (n-1)/2.  If n is even = 2*j, this is (2*j-1)/2 = j-1/2 so
    # j-1 is the largest, which is n//2 - 1.  If n is odd = 2*j+1, this is
    # (2*j+1-1)/2 = j so j-1 is the largest, and that's again n//2-1.
	

def merge(*iterables, key=None, reverse=False):
    '''Merge multiple sorted inputs into a single sorted output.

    Similar to sorted(itertools.chain(*iterables)) but returns a generator,
    does not pull the data into memory all at once, and assumes that each of
    the input streams is already sorted (smallest to largest).

    >>> list(merge([1,3,5,7], [0,2,4,8], [5,10,15,20], [], [25]))
    [0, 1, 2, 3, 4, 5, 5, 7, 8, 10, 15, 20, 25]

    If *key* is not None, applies a key function to each element to determine
    its sort order.

    >>> list(merge(['dog', 'horse'], ['cat', 'fish', 'kangaroo'], key=len))
    ['dog', 'cat', 'fish', 'horse', 'kangaroo']

    '''
    """多个列表合并为一个最小堆，为生成器"""
    
def nsmallest(n, iterable, key=None):
    """Find the n smallest elements in a dataset.

    Equivalent to:  sorted(iterable, key=key)[:n]
    """
    """找到最小的几个元素"""
    
def nlargest(n, iterable, key=None):
    """Find the n largest elements in a dataset.

    Equivalent to:  sorted(iterable, key=key, reverse=True)[:n]
    """
	"""找到最大的几个元素"""
```



##### 4. 举例说明

```python
>>> nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
>>> heap = list(nums)
>>> heapq.heapify(heap)
>>> heap
[-4, 2, 1, 23, 7, 2, 18, 23, 42, 37, 8]
>>> heapq.heappush(heap, 15)
>>> heap
[-4, 2, 1, 23, 7, 2, 18, 23, 42, 37, 8, 15]
>>> heapq.heappush(heap, 13)
>>> heap
[-4, 2, 1, 23, 7, 2, 18, 23, 42, 37, 8, 15, 13]
>>> 
>>> heapq.heappush(heap, -1)
>>> heap
[-4, 2, -1, 23, 7, 2, 1, 23, 42, 37, 8, 15, 13, 18]
>>> heapq.heappop(heap)
-4
>>> heapq.heappop(heap)
-1
>>> heap
[1, 2, 2, 23, 7, 13, 18, 23, 42, 37, 8, 15]
>>> heapq.heapreplace(heap, 22)
1
>>> heap
[2, 2, 13, 23, 7, 15, 18, 23, 42, 37, 8, 22]
>>> heapq.heappushpop(heap, 14)
2
>>> heap
[2, 7, 13, 23, 8, 15, 18, 23, 42, 37, 14, 22]

>>> nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
>>> heapq.nsmallest(3, nums)
[-4, 1, 2]
>>> heapq.nlargest(3, nums)
[42, 37, 23]
```

