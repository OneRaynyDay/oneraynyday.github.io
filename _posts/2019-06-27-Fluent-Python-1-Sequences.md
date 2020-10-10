---
published: true
title: Python Fundamentals - Sequences
use_math: false
category: dev
layout: default
---

# Table of Contents

* TOC
{:toc}

# Overview

In python's standard library, we have the following sequence types:

- `list`. Heterogenous and mutable.
- `tuple`. Heterogenous and immutable.
- `collections.deque`. Heterogenous and mutable.
- `str`. Flat and immutable.
- `bytes`. Flat and immutable.
- `bytearray`. Flat and mutable.
- `memoryview`. Flat and mutable.
- `array.array`. Flat and mutable.

A data structure is **flat/heterogenous** if it can only hold the same type or hold different types, and is **mutable/immutable** if its contents can be modified.

Listcomps are fairly simple, so I will just give a few examples of them here before we jump to genexps.

### Cartesian Products
```python
>>> letters = ['a','b','c']
>>> numbers = [1,2,3]
>>> product = [(letter, number) for letter in letters for number in numbers]
>>> product
[('a', 1), ('a', 2), ('a', 3), ('b', 1), ('b', 2), ('b', 3), ('c', 1), ('c', 2), ('c', 3)]
```

### Restricted listcomp
```python
>>> letters = ['a','b','c']
>>> filtered_letters = [letter for letter in letters if letter != 'c']
>>> filtered_letters
['a', 'b']
```

# List & Tuples & Generic Sequences

## Generator Expressions (genexp)

Generator expressions can be done to save space in the case that an entire list is not needed. We replace listcomp with genexp in the above cartesian product example:

```python
>>> letters = ['a','b','c']
>>> numbers = [1,2,3]
>>> for element in ('%s %s' % (l, n) for l in letters for n in numbers):
...		print(element)
...
a 1
a 2
a 3
b 1
b 2
b 3
c 1
c 2
c 3
```

Here, we did not create a cartesian product list and scan it to print. This is clearly more efficient.

## `tuple`

Tuples are immutable lists which serve as records with no field names. In particular, the position of the item in the tuple may have a semantic meaning. We unpack a tuple like so:

```python
>>> coordinates = (1,2)
>>> _, longitude = coordinates
>>> longitude
2
```

Another way to unpack is by using the star operator:

```python
>>> coordinates = (1,2)
>>> f = lambda x, y: x+y
>>> f(*coordinates)
3
```

You can use these two in mixture:

```python
>>> long_tup = (1,2,3,4,5,6,7)
>>> x,y,*z = long_tup
>>> x
1
>>> y
2
>>> z
[3, 4, 5, 6, 7]
```

### `namedtuple`

`namedtuple`s are a part of the `collections` package which basically, as the name implies, have names to each field. Just as we create tuples as nameless records, we have `namedtuples` for records with names. These should be used more often:

```python
>>> from collections import namedtuple
>>> Snek = namedtuple('Snake', ['name', 'length'])
>>> an = Snek('anaconda', '4.511')
>>> py = Snek('python', '3.7')
>>> py
Snake(name='python', length='3.7')
>>> an
Snake(name='anaconda', length='4.511')
>>> py[1]
'3.7'
```

## Advanced Slicing

### Slice Assignment

You can modify portions of a **mutable** sequence using slice, i.e. add, delete, etc:

#### Add

```python
>>> l = [1,2,3]
>>> l[1:1] = ['a','b','c']
>>> l
[1, 'a', 'b', 'c', 2, 3]
```

#### Delete

```python
>>> l = [1,2,3]
>>> del l[1:]
>>> l
[1]
```

### No More Magic - Readable Slices

When you call `seq[start:stop:step]`, python calls `seq.__getitem__(slice(start, stop, step))`, where `slice` is actually an object. Thus, you can actually create **named constant slices** for better readability, like so:

```python
>>> NAME = slice(0,4)
>>> FIRST = slice(4,10)
>>> LAST = slice(10, None)
>>> items
['a   123   456', 'b   23    4567']
>>> for item in items:
...     print("Name: {}, First: {}, Last: {}".format(item[NAME].strip(), item[FIRST].strip(), item[LAST].strip()))
...
Name: a, First: 123, Last: 456
Name: b, First: 23, Last: 4567
```

This makes your functions less "magic" and more readable for future developers.

### Multi-Argument Slices - How?

When you use `numpy`'s multidimensional array class, `np.ndarray`, how does it handle multiple arguments?

When you call `a[i,j]`, you are actually calling `a.__getitem__((i, j))`, so that the arguments inside of `[]` is packaged as a tuple. `numpy` defined custom `__getitem__` so it can accept tuples like above, unlike built-in types.

When you use `a[i, ...]`, it is another multi-argument slice, equivalent to `a[i, :, :, :]` if it was a 4-dimensional matrix. The `...` is considered a special keyword, an alias to `Ellipsis` object, from the `ellipsis` class. Think of `Ellipsis` as `True` or `False`, and its class `ellipsis` like `bool`.

## `+, *` Operators With Sequences

### Reference Issues with `*`

**Never do this:** `[[]] * 3`. When you evaluate it at first, you'll see `[[],[],[]]`. Innocent looking enough, but they are references to the same list. **Do this instead:** `[[] for _ in range(3)]`, so that each object is constructed individually. 

### `__iadd__` and `__imul__` With Immutable Objects

When you try to augment a sequence using `+=` or `*=`, it invokes the magic functions `__iadd__` and `__imul__`. If the object does not have these functions, it falls back to `__add__` and `__mul__`. For immutable objects like tuples, this sometimes spells trouble:

```python
>>> l = (1,2,3)
>>> id(l)
4510772608
>>> l += (4,5,6)
>>> l
(1, 2, 3, 4, 5, 6)
>>> id(l)
4510680840
```

**These are not the same objects.** What's happening underneath the hood is equivalent to `l = l + (4,5,6)`, which means we created a new tuple! **To be precise, the tuple container is replaced while the references to objects inside are the same before and after.**

### Weirdest Corner Case

```python
>>> t = ([],)
>>> t[0] += [1]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> t
([1],)
```

Wait... So we modified the list inside the tuple, but it also gave us an error? Why is this true? **Because this is not an atomic operation.** Specifically, it looks like this:

```python
>>> temp_t = t
>>> temp_idx = 0
>>> mutable_item = temp_t[temp_idx]
>>> mutable_item += [1]
>>> temp_t[temp_idx] = mutable_item
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

You can also disassemble python code and view the bytecode to see a similar process. 

## `bisect` For Sorted Sequences

### Searching Using `bisect()`

There are 2 bisect functions, `bisect_left` and `bisect_right`. By default, `bisect.bisect` is an alias for `bisect.bisect_right`. These functions are basically binary searches for sequences with generic ordered types. The difference between the two is subtle - when you try to search an object and the object itself is present in the sequence, `bisect_left` returns the element's index, and `bisect_right` returns the index plus one:

```python
>>> from bisect import bisect_right, bisect_left
>>> l = [1,2,3]
>>> bisect_right(l, 2)
2
>>> bisect_left(l, 2)
1
```

One application is **discretization**. For example, from numerical scores on an exam to one's final grade, which is in discrete intervals of `F, D, C, B, A`. Or, waist size to shirt sizes:

```python
>>> def shirt_size(size, cutoff=[30,60], sizes=['Small', 'Medium', 'Large']):
...     return sizes[bisect_right(cutoff, size)]
...
>>> shirt_size(20)
'Small'
>>> shirt_size(30)
'Medium'
>>> shirt_size(50)
'Medium'
>>> shirt_size(60)
'Large'
>>> shirt_size(100)
'Large'
```

Here, we use `bisect_right` because we would rather want the person to fit comfortably in a larger t-shirt in case their waist size was at the cutoff. If you want to look like you're bigger than you actually are(like me), then you would use `bisect_left`.

### Inserting Using `insort()`

We use `bisect.insort` to add an element into a sorted sequence:

```python
>>> import bisect
>>> l = [1,3,5,7]
>>> for i in range(4):
...     bisect.insort(l, i*2)
...     print(l)
...
[0, 1, 3, 5, 7]
[0, 1, 2, 3, 5, 7]
[0, 1, 2, 3, 4, 5, 7]
[0, 1, 2, 3, 4, 5, 6, 7]
```

`insort` has extra keyword arguments to insert in a sorted subsequence, and insort left or right like bisect. Once again, this can be applied to more than just lists. Any ordered collection will do, like `array`s!

# Homogenous Array Types

## `array`

**Arrays are underrated and underused. They are so, so much faster than `list`s. `list`s are the default, but don't be lazy when you need the performance.** `array`s contain the bit & byte level representation of primitive data types, and it's basically a C-style array. To create an array:

```python
>>> from array import array
>>> import numpy as np
>>> nums = array('d', (np.random.random() for _ in range(10)))
>>> for num in nums:
...     print(num)
...
0.2076318634616442
0.5052559930909137
0.26556051714640794
0.3538229563850064
0.24394891007765362
0.829697244498978
0.8050680531932854
0.7540974416748557
0.5157377814111441
0.6025949390048687
>>> with open('temp.bin', 'wb') as f:
...     nums.tofile(f)
...
```

And then we see the saved binary file:

```
ls -la temp.bin
-rw-r--r--  1 dev  staff  80 Jun 27 16:18 temp.bin
```

Nice. It's small and compact in binary format. `numpy` arrays do something similar. `bytes` and `bytearray` are simply specific types of `array` that will be discussed in detail later. 

## `memoryview`

`memoryview` is like a slice of an `array`. There is no copying, everything is referenced and is usually mutable. Here we change the content of the first double in an `array` by casting the memoryview to unsigned 8-bit ints, and modifying it:

```python
>>> from array import array
>>> nums = array('d', [1,2,3])
>>> mv = memoryview(nums)
>>> mvc = mv.cast('B')
>>> mvc.tolist()
[0, 0, 0, 0, 0, 0, 240, 63, 0, 0, 0, 0, 0, 0, 0, 64, 0, 0, 0, 0, 0, 0, 8, 64]
>>> mvc[0] = 10
>>> nums
array('d', [1.0000000000000022, 2.0, 3.0])
```

Most of the time, you probably don't want to use this though.

# Deques & Queues

## `deque`

`collections` supplies with us a `deque` container which is heterogenous. It has the standard `append` and `pop` operations. On top of it, it has a couple cool functions like `rotate`, `extend` and `extendleft`.

`rotate()` takes in a single integer as argument, and rotates the deque in that direction. Why no `rotateleft` like `extend`? Because you can supply a negative number to rotate in the other direction. By default, `rotate(k)` moves the i-th element to the `(i+k) % N`-th place in the deque, where `N` is the size of the container.

```python
>>> from collections import deque
>>> dq = deque("abcdefg", maxlen=4)
>>> dq
deque(['d', 'e', 'f', 'g'], maxlen=4)
>>> dq.rotate(2)
>>> dq
deque(['f', 'g', 'd', 'e'], maxlen=4)
>>> dq.rotate(-1)
>>> dq
deque(['g', 'd', 'e', 'f'], maxlen=4)
>>> dq.extend("abc")
>>> dq
deque(['f', 'a', 'b', 'c'], maxlen=4)
>>> dq.extendleft("gfe")
>>> dq
deque(['e', 'f', 'g', 'f'], maxlen=4)
```

## Different Queues

There are a lot of different queue containers in python. You can technically use a `deque` as a queue itself too. However, for more complicated applications where asynchronicity is a key factor, insertion and popping from a queue may be tricky. Here, we use threadsafe `queue` library with its `Queue`, `LifoQueue` (which is literally a stack), and `PriorityQueue`. When you call `pop` on an empty default queue here, it will wait until an item has been inserted, rather than return an error message. `multiprocessing` and `asyncio` implements its own queues as well.
