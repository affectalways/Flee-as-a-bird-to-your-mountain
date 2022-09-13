# Python计算内存时需要注意的问题

##### 目录

- [问题](#问题)
- [原因](#原因)
- [解决方案](#解决方案)



#### 问题

一般使用`sys.getsizeof()`来计算内存，但是用这个方法计算时，可能会出现意料不到的问题。

python官方文档中关于这个方法的介绍有两层意思：

- 该方法用于获取一个对象的字节大小（bytes）
- 它只计算直接占用的内存，而不计算对象内所引用对象的内存

也就是说，getsizeof() 并不是计算实际对象的字节大小，而是计算“占位对象”的大小。如果你想计算所有属性以及属性的属性的大小，getsizeof() 只会停留在第一层，这对于存在引用的对象，计算时就不准确。

例如列表 [1,2]，getsizeof() 不会把列表内两个元素的实际大小算上，而只是计算了对它们的引用。

举一个形象的例子，我们把列表想象成一个箱子，把它存储的对象想象成一个个球，现在箱子里有两张纸条，写上了球 1 和球 2 的地址（球不在箱子里），getsizeof() 只是把整个箱子称重（含纸条），而没有根据纸条上地址，找到两个球一起称重。

![image-20220913214836008](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/image-20220913214836008.png)

如图所示，单独计算 a 和 b 列表的结果是 72 和 120，然后把它们作为 c 列表的子元素时，该列表的计算结果却仅仅才 72。

就是说：getsizeof() 方法在计算列表大小时，其结果跟元素个数相关，但跟元素本身的大小无关。

下面再看看字典的例子：

![]()![Python计算内存时需要注意的问题1](https://raw.githubusercontent.com/affectalways/Flee-as-a-bird-to-your-mountain/main/img/Python%E8%AE%A1%E7%AE%97%E5%86%85%E5%AD%98%E6%97%B6%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E7%9A%84%E9%97%AE%E9%A2%981.jpeg)

明显可以看出，三个字典实际占用的全部内存不可能相等，但是 getsizeof() 方法给出的结果却相同，这意味着它只关心键的数量，而不关心实际的键值对是什么内容，情况跟列表相似。

## 

#### 原因

有个概念叫“浅拷贝”，指的是 copy() 方法只拷贝引用对象的内存地址，而非实际的引用对象。类比于这个概念，我们可以认为 getsizeof() 是一种“浅计算”。

“浅计算”不关心真实的对象，所以其计算结果只是一个假象。这是一个值得注意的问题，但是注意到这点还不够，我们还可以发散地思考如下的问题：

- “浅计算”方法的底层实现是怎样的？
- 为什么 getsizeof() 会采用“浅计算”的方法？

关于第一个问题，getsizeof(x) 方法实际会调用 x 对象的`__sizeof__()` 魔术方法，对于内置对象来说，这个方法是通过 CPython 解释器实现的。

我查到这篇文章《Python中对象的内存使用(一)》，它分析了 CPython 源码，最终定位到的核心代码是这一段：

```c
/*longobject.c*/

static Py_ssize_t
int___sizeof___impl(PyObject *self)
{
    Py_ssize_t res;

    res = offsetof(PyLongObject, ob_digit) + Py_ABS(Py_SIZE(self))*sizeof(digit);
    return res;
}
```

我看不懂这段代码，但是可以知道的是，它在计算 Python 对象的大小时，只跟该对象的结构体的属性相关，而没有进一步作“深度计算”。

对于 CPython 的这种实现，我们可以注意到两个层面上的区别：

- 字节增大：int 类型在 C 语言中只占到 4 个字节，但是在 Python 中，int 其实是被封装成了一个对象，所以在计算其大小时，会包含对象结构体的大小。在 32 位解释器中，getsizeof(1) 的结果是 14 个字节，比数字本身的 4 字节增大了。
- 字节减少：对于相对复杂的对象，例如列表和字典，这套计算机制由于没有累加内部元素的占用量，就会出现比真实占用内存小的结果。

由此，我有一个不成熟的猜测：基于“一切皆是对象”的设计原则，int 及其它基础的 C 数据类型在 Python 中被套上了一层“壳”，所以需要一个方法来计算它们的大小，也即是 getsizeof()。

官方文档中说“All built-in objects will return correct results” [1]，指的应该是数字、字符串和布尔值之类的简单对象。但是不包括列表、元组和字典等在内部存在引用关系的类型。

为什么不推广到所有内置类型上呢？我未查到这方面的解释，若有知情的同学，烦请告知。



#### 解决方案

​		与浅拷贝相对应，还有一种深拷贝。对于前面的两个例子，深拷贝应该遍历每个内部元素以及可能的子元素，累加计算它们的字节，最后算出总的内存大小。

那么，我们应该注意的问题有：

- 是否存在深拷贝的方法/实现方案？
- 实现深拷贝时应该注意什么？

Stackoverflow 网站上有个年代久远的问题[“How do I determine the size of an object in Python?”](https://stackoverflow.com/questions/449560/how-do-i-determine-the-size-of-an-object-in-python) 实际上问的就是如何实现深拷贝的问题。

The [Pympler](https://pypi.python.org/pypi/Pympler) package's `asizeof` module can do this.

Use as follows:

```py
from pympler import asizeof
asizeof.asizeof(my_object)
```

Unlike `sys.getsizeof`, it **works for your self-created objects**. It even works with numpy.

```py
>>> asizeof.asizeof(tuple('bcd'))
200
>>> asizeof.asizeof({'foo': 'bar', 'baz': 'bar'})
400
>>> asizeof.asizeof({})
280
>>> asizeof.asizeof({'foo':'bar'})
360
>>> asizeof.asizeof('foo')
40
>>> asizeof.asizeof(Bar())
352
>>> asizeof.asizeof(Bar().__dict__)
280
>>> A = rand(10)
>>> B = rand(10000)
>>> asizeof.asizeof(A)
176
>>> asizeof.asizeof(B)
80096
```

As [mentioned](https://stackoverflow.com/questions/449560/how-do-i-determine-the-size-of-an-object-in-python/33631772?noredirect=1#comment85591087_33631772),

> [The (byte)code size of objects like classes, functions, methods, modules, etc. can be included by setting option `code=True`.](https://pythonhosted.org/Pympler/asizeof.html#asizeof)

And if you need other view on live data, Pympler's

> module [`muppy`](https://pythonhosted.org/Pympler/muppy.html#muppy) is used for on-line monitoring of a Python application and module [`Class Tracker`](https://pythonhosted.org/Pympler/classtracker.html#classtracker) provides off-line analysis of the lifetime of selected Python objects.



但是，`pympler`不适合大数据量内存计算，会报错，但是可以通过切片的方式粗略计算内存占用大小！



#### 参考链接

- [https://stackoverflow.com/questions/449560/how-do-i-determine-the-size-of-an-object-in-python](https://stackoverflow.com/questions/449560/how-do-i-determine-the-size-of-an-object-in-python)
- [https://cloud.tencent.com/developer/article/1594988](https://cloud.tencent.com/developer/article/1594988)