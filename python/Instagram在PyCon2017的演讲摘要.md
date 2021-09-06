---
title: "Instagram在PyCon2017的演讲摘要"
date: 2020-07-21T22:38:37+08:00
tags: ["python"]
keywords: 
- python
- blog
- 博客
- Instagram
- Instagram在PyCon2017的演讲摘要
- pycon
- 算法
description: "python，Instagram在PyCon2017的演讲摘要"
categories: ["python"]
draft: false
---

**郑重声明：本篇文章非原创，摘自<https://www.zlovezl.cn/articles/instagram-pycon-2017/>**



![图：Instagram Loves Python](http://www.zlovezl.cn/static//uploaded/2017/05/2017-05-30-22-31-37_thumb.jpg)





## PyCon 简介

PyCon 是全世界最大的以 [Python 编程语言](https://en.wikipedia.org/wiki/Python_(programming_language)) 为主题的技术大会。大会由 Python 社区组织，每年举办一次。在大会上，来自世界各地的 Python 用户与核心开发者齐聚一堂，共同分享 Python 世界的新鲜事、Python 语言的应用案例、使用技巧等等内容。



## Instagram 简介

Instagram 是一款移动端的照片与视频分享软件，由 Kevin Systrom 和 Mike Krieger 在 2010 年创办。Instagram 在发布后开始快速流行。于 2012 年被 Facebook 以 10 亿美元的价格收购。而当时 Instagram 的员工仅有区区 13 名。

如今，**Instagram 的总注册用户达到 30 亿，月活用户超过 7 亿** *（作为对比，微信最新披露的月活跃用户为 9.38 亿）*。而令人吃惊的是，这么高的访问量背后，竟完全是由以速度慢著称的 Python + Django 支撑。

在 Python 2017 上，Instagram 的工程师们带来了一个有关 Python 在 Instagram 的主题演讲，同时还分享了 Instagram 如何将整个项目运行环境升级到 Python 3 的故事。

本文为该次演讲的内容摘要。



## Python @Instagram

### 为什么选择 Python 和 Django

Instagram 选择 Django 的原因很简单，Instagram 的两位创始人 *(Kevin Systrom and Mike Krieger)* 都是产品经理出身。**在他们想要创造 Instagram 时，Django 是他们所知道的最稳定和成熟的技术之一。**

时至今日，即使已经拥有超过 30 亿的注册用户。Instagram 仍然是 Python 和 Django 的重度使用者。Instagram 的工程师 Hui Ding 说到： *『一直到用户 ID 已经超过了 32bit int 的限额（约为 20 亿），Django 本身仍然没有成为我们的瓶颈所在。』*

不过，除了使用 Django 的原生功能外，Instagram 还对 Django 做了很多定制化工作：

- 扩展 Django Models 使其支持 Sharding *（一种数据库分片技术）*，[Instagram Engneering](https://engineering.instagram.com/@InstagramEng) 博客专门为这件事情写过一篇博客，可参阅：[Sharding & IDs at Instagram](https://engineering.instagram.com/sharding-ids-at-instagram-1cf5a71e5a5c)

- 手动关闭 GC（垃圾回收）来提升 Python 内存管理效率，他们同样也写过一篇博客来说明这件事情：[Dismissing Python Garbage Collection at Instagram](https://engineering.instagram.com/dismissing-python-garbage-collection-at-instagram-4dca40b29172)

- 在位于不同地理位置的多个数据中心部署整套系统

  

### Python 语言的优势所在

Instagram 的联合创始人 Mike Krieger 说过： *『我们的用户根本不关心 Instagram 使用了哪种关系数据库，他们当然也不关心 Instagram 是用什么编程语言开发的。』*

所以，Python 这种 **简单** 而且 **实用至上** 的编程语言最终赢得了 Instagram 的青睐。他们认为，使用 Python 这种简单的语言有助于塑造 Instagram 的工程师文化，那就是：

1. **专注于定位问题、解决问题** - 而不是工具本身的各种花花绿绿的特性
2. **使用那些经过市场验证过的成熟技术方案** - 而不用被工具本身的问题所烦扰
3. **用户至上：专注于用户所能看到的新特性，为用户带去价值**

但是，即使使用 Python 语言有这么多好处，它还是很慢，不是吗？

不过，这对于 Instagram 不是问题，因为他们认为：**『Instagram 的最大瓶颈在于开发效率，而不是代码的执行效率』**

> At Instagram, our bottleneck is development velocity, not pure code execution.

所以，最终的结论是：**你完全可以使用 Python 语言来实现一个超过几十亿用户使用的产品，而根本不用担心语言或框架本身的性能瓶颈。**



### 如何提升运行效率

但是，即使是选用了拥有诸多好处的 Python 和 Django。在 Instagram 的用户数迅速增长的过程中，性能问题还是出现了：**服务器数量的增长率已经慢慢的超过了用户增长率**。Instagram 是怎么应对这个问题的呢？

他们使用了这些手段来缓解性能问题：

- **开发工具来帮助调优**：Instagram 开发了很多涵盖各个层面的工具，来帮助他们进行性能调优以及找到性能瓶颈。
- **使用 C/C++ 来重写部分组件**：把那些稳定而且对性能最敏感的组件，使用 C 或 C++ 来重写，比如访问 memcache 的 library。
- **使用 Cython**：Cython 也是他们用来提升 Python 效率的法宝之一。

除了上面这些手段，他们还在探索异步 IO 以及新的 Python Runtime 所能带来的性能可能性。



### 升级到 Python 3

在相当长的一段时间，Instagram 都跑在 Python 2.7 + Django 1.3 的组合之上。在这个已经落后社区很多年的环境上，他们的工程师们还打了非常非常多的小 patch。难道他们要被永远卡在这个版本上吗？

所以，在经过一系列的讨论后，他们最终做出一个重大的决定：**升级到 Python 3！！**

事实上，Instagram 目前已经完成了将运行环境迁移到 Python 3 的工作 - 他们的整套服务已经在 Python 3 上跑了好几个月了。那么他们是怎么做到的呢？接下来便是由 Instagram 工程师 Lisa guo 带来的 Instagram 如何迁移到 Python 3 的故事。



## Instagram 升级到 Python 3 的故事

### 为什么要升级到 Python 3

对于 Instagram 来说，下面这些因素是推动他们将运行环境迁移到 Python 3 的主要原因：

#### 1. 新特性：类型注解 Type Annotations

看看下面这段代码：

```
def compose_from_max_id(max_id):
    '''@param str max_id'''
```

图中函数的 `max_id` 参数究竟是什么类型呢？int？tuple？或是 list? 等等，函数文档里面说它是 str 类型。

但随着时间推移，万一这个参数的类型发生变化了呢？如果某位粗心的工程师修改代码的同时忘了更新文档，那就会给函数的使用者带来很大麻烦，最终还不如没有注释呢。

#### 2. 性能

Instagram 的整个 Django Stack 都跑在 uwsgi 之上，全部使用了同步的网络 IO。这意味着同一个 uwsgi 进程在同一时间只能接收并处理一个请求。这让如何调优每台机器上应该运行的 uwsgi 进程数成了一个麻烦事：

*为了更好利用 CPU，使用更多的进程数？但那样会消耗大量的内存。而过少的进程数量又会导致 CPU 不能被充分利用。*

为此，他们决定跳过 Python 2 中哪些蹩脚的异步 IO 实现 *（可怜的 gevent、tornado、twisted 众）*，直接升级到 Python 3，去探索标准库中的 asyncio 模块所能带来的可能性。

#### 3. 社区

因为 Python 社区已经停止了对 Python 2 的支持。如果把整个运行环境升级到 Python 3，Instagram 的工程师们就能和 Python 社区走的更近，可以更好的把他们的工作回馈给社区。



### 确定迁移方案

在 Instagram，进行 Python 3 的迁移需要必须满足两个前提条件：

1. 不停机，不能有任何的服务因此不可用
2. 不能影响产品新特性的开发

但是，在 Instagram 的开发环境中，要满足上面这两点来完成迁移到 Python 3.6 这种庞大的工程是非常困难的。



#### 基于主分支的开发流程

即便使用了以多分支功能著称的 git，Instagram 所有的开发工作都是主要在 master 分支上进行的，Instagram 所奉行的开发哲学是：**『不管是多大的新特性或代码重构，都应该拆解成较小的 Commit 来进行。』**

那些被合并进 master 分支的代码，都将在一个小时内被发布到线上环境。**而这样的发布过程每天将会发生上百次。**在这么频繁的发布频率下，如何在满足之前的那两个前提下来完成迁移变得尤其困难。



#### 被弃用的迁移方案

**创建一个新分支**

很多人在处理这类问题时，第一个蹦进脑子的想法就是： *『让我们创建一个分支，当我们开发完后，再把分支合并进来』*

但在 Instagram 这么高的迭代频率上，使用一个独立分支并不是好主意：

1. Instagram 的 Codebase 每天都在频繁更新，在开发 Python 3 分支的过程中，让新分支与现有 master 分支保持同步开销极大，同时极易出错
2. 最终将 Python 3 分支这个改动非常多的分支合并回 Master 拥有非常高的风险
3. 只有少数几个工程师在 Python 3 分支上专职负责升级工作，其他想帮助迁移工作的工程师无法参与进来

**挨个替换接口**

还有一个方案就是，挨个替换 Instagram 的 API 接口。但是 Instagram 的不同接口共享着很多通用模块。这个方案要实施起来也非常困难。

**微服务**

还有一个方案就是将 Instagram 改造成微服务架构。通过将那些通用模块重写成 Python 3 版本的微服务来一步步完成迁移工作。

但是这个方案需要重新组织海量的代码。同时，当发生在进程内的函数调用变成 RPC 后 ，整个站点的延迟会变大。此外，更多的微服务也会引入更高的部署复杂度。

所以，既然 Instagram 的开发哲学是：**小步前进，快速迭代**。他们最终决定的方案是：**一步一步来，最终让 master 分支上的代码同时兼容 Python 2 和 Python 3 。**



### 开始迁移工作

既然要让整个 codebase 同时兼容 Python 2 和 Python 3，那么首先要符合这点的就是那些被大量使用的第三方 package。针对第三方 package，Instagram 做到了下面几点：

- 拒绝引入所有不兼容 Python 3 的新 package
- 去掉所有不再使用的 package
- 替换那些不兼容 Python 3 的 package

在代码的迁移过程中，他们使用了工具 [modernize](https://python-modernize.readthedocs.io/en/latest/) 来帮助他们。

使用 modernize 时，有一个小技巧：**每次修复多个文件的一个兼容问题，而不是一下修复一个文件中的多个兼容问题。** 这样可以让 Code Review 过程简单很多，因为 Reviewer 每次只需要关注一个问题。



### 使用单元测试来帮助迁移

对于 Python 这种灵活性极强的动态语言来说，除了真正去执行代码外，几乎没有其他比较好的检查代码错误的手段。

前面提到，Instagram 所有被合并到 master 的代码提交会在一个小时内上线到线上环境，但这不是没有前提条件的。**在上线前，所有的提交都需要通过成千上万个单元测试。**

于是，他们开始加入 Python 3 来执行所有的单元测试。一开始，只有极少数的单元测试能够在 Python 3 环境下通过，但随着 Instagram 的工程师们不断的修复那些失败的单元测试，最终所有的单元测试都可以在 Python 3 环境下成功执行。



#### 单元测试的局限性

但是，单元测试也是有局限性的：

- Instagram 的单元测试没有做到 100% 的代码覆盖率
- 很多第三方模块都使用了 mock 技术，而 mock 的行为与真实的线上服务可能会有所不同

所以，当所有的单元测试都被修复后，他们开始在线上正式使用 Python 3 来运行服务。

这个过程并不是一蹴而就的。首先，所有的 Instagram 工程师开始访问到这些使用 Python 3 来执行的新服务，然后是 Facebook 的所有雇员，随后是 0.1%、20% 的用户，最终 Python 3 覆盖到了所有的 Instagram 用户。

![图：循序渐进的发布流程](http://www.zlovezl.cn/static//uploaded/2017/05/2017-05-29-09-35-43_thumb.jpg)





### 迁移过程的技术问题

Instagram 在迁移到 Python 3 时碰到很多问题，下面是最典型的几个：

#### Unicode 相关的字符串问题

Python 3 相比 Python 2 最大的改动之一，就是在语言内部对 unicode 的处理。

在 Python 2 中，文本类型 *（也就是 unicode）* 和二进制类型 *（也就是 str）* 的边界非常模糊。很多函数的参数既可以是文本，也可以是二进制。但是在 Python 3 中，文本类型和二进制类型的字符串被完全的区分开了。

于是，下面这段在 Python 2 下可以正常运行的代码在 Python 3 下就会报错：

```
mymac = hmac.new('abc')
TypeError: key: expected bytes or bytearray, but got 'str'
```

解决办法其实很简单，只要加上判断：如果 value 是文本类型，就将其转换为二进制。如下所示：

```
value = 'abc'
if isinstance(value, six.text_type):
    value = value.encode(encoding='utf-8')
mymac = hmac.new(value)
```

但是，在整个代码库中，像上面这样的情况非常多。作为开发人员，如果需要在调用每个函数时都要想想： *这里到底是应该编码成二进制，或者是解码成文本呢？* 将会是非常大的负担。

于是 Instagram 封装了一些名为 `ensure_str()`、`ensure_binary()`、`ensure_text()` 的帮助函数，开发人员只需对那些不确定类型的字符串，使用这些帮助函数先做一次转换就好。

```
mymac = hmac.new(ensure_binary('abc'))
```

#### 不同 Python 版本的 pickle 差异

Instagram 的代码中大量使用了 pickle。比如用它序列化某个对象，然后将其存储在 memcache 中。如下面的代码所示：

```
memcache_data = pickle.dumps(data, pickle.HIGHEST_PROTOCOL)
data = pickle.loads(memcache_data)
```

问题在于，Python 2 与 Python 3 的 pickle 模块是有差别的。

如果上文的第一行代码，刚好是由 Python 3 运行的服务进行序列化后存入 memcache。而反序列化的过程却是由 Python 2 进行，那代码运行时就会出现下面的错误：

```
ValueError: unsupported pickle protocol: 4
```

这是由于在 Python 3 中，`pickle.HIGHEST_PROTOCOL` 的值为 `4`，而 Python 2 中的的 pickle 最高支持的版本号却是 `2`。那么如何解决这个问题呢？

Instagram 最终选择让 Python 2 和 Python 3 使用完全不同的 namespace 来访问 memcache。通过将二者的数据读写完全隔开来解决这个问题。

#### 迭代器

在 Python 3 中，很多内置函数被修改成了只返成迭代器 Iterator：

```
map()
filter()
dict.items()
```

迭代器有诸多好处，最大的好处就是，使用迭代器不需要一次性分配大量内存，所以它的内存效率比较高。

但是迭代器有一个天然的特点，当你对某个迭代器做了一次迭代，访问完它的内容后，就没法再次访问那些内容了。**迭代器中的所有内容都只能被访问一次。**

在 Instagram 的 Python 3 迁移过程中，就因为迭代器的这个特性被坑了一次，看看下面这段代码：

```
CYTHON_SOURCES = [a.pyx, b.pyx, c.pyx]
builds = map(BuildProcess, CYTHON_SOURCES)
while any(not build.done() for build in builds):
    pending = [build for build in builds if not build.started()]
    <do some work>
```

这段代码的用处是挨个编译 Cython 源文件。当他们把运行环境切换到 Python 3 后，一个奇怪的问题出现了：**CYTHON_SOURCES 中的第一个文件永远都被跳过了编译。**为什么呢？

这都是迭代器的锅。在 Python 3 中，`map()` 函数不再返回整个 list，而是返回一个迭代器。

于是，当第二行代码生成 builds 这个迭代器后，第三行代码的 while 循环迭代了 builds，刚好取出了第一个元素。**于是之后的 pending 对象便里面永远少了那第一个元素。**

这个问题解决起来也挺简单的，你只要手动的吧 builds 转换成 list 就可以了：

```
builds = list(map(BuildProcess, CYTHON_SOURCES))
```

但是这类 bug 非常难定位到。如果用户的 feeds 里面永远少了那最新的第一条，用户很少会注意到。

#### 字典的顺序

看看下面这段代码：

```
>>> testdict = {'a': 1, 'b': 2, 'c': 3}
>>> json.dumps(testdict)
```

它会输出什么结果呢？

```
# Python2
'{"a": 1, "c": 3, "b": 2}'
# Python 3.5.1
'{"c": 3, "b": 2, "a": 1}'    # or
'{"c": 3, "a": 1, "b": 2}'
# Python 3.6
'{"a": 1, "b": 2, "c": 3}'
```

在不同的 Python 版本下，这个 json dumps 的结果是完全不一样的。甚至在 3.5.1 中，它会完全随机的返回两个不同的结果。Instagram 有一段判断配置文件是否发生变动的模块，就是因为这个原因出了问题。

这个问题的解决办法是，在调用 `json.dumps` 传入 `sort_keys=True` 参数：

```
>>> json.dumps(testdict, sort_keys=True)
'{"a": 1, "b": 2, "c": 3}'
```

### 迁移到 Python 3.6 后的性能提升

当 Instagram 解决了这些奇奇怪怪的版本差异问题后，还有一个巨大的谜题困扰着他们：**性能问题**。

在 Instagram，他们使用两个主要指标来衡量他们的服务性能：

- 每次请求产生的 CPU 指令数（越低越好）
- 每秒能够处理的请求数（越高越好）

所以，当所有的迁移工作完成后，他们非常惊喜的发现：**第一个性能指标，每次请求产生的 CPU 指令数居然足足下降了 12% ！！！**

但是，按理说第二个指标 - 每秒请求数也应该获得接近 12% 的提升。不过最后的变化却是 0%。究竟是出了什么问题呢？

他们最终定位到，是由于不同 Python 版本下的内存优化配置不同，导致 CPU 指令数下降带来的性能提升被抵消了。那为什么不同 Python 版本下的内存优化配置会不一样呢？

这是他们用来检查 uwsgi 配置的代码：

```
if uwsgi.opt.get('optimize_mem', None) == 'True':
    optimize_mem()
```

注意到那段 `... ... == 'True'` 了吗？在 Python 3 中，这个条件判断总是不会被满足。问题就在于 unicode。在将代码中的 `'True'` 换成 `b'True'`（也就是将文本类型换成二进制，这种判断在 Python 2 中完全不区分的）后，问题解决了。

**所以，最终因为加上了一个小小的字母 'b'，程序的整体性能提升了 12%。**

### 结论

在今年二月份，Instagram 的后端代码的运行环境完全切换到了 Python 3 下：

![图：Instagram 版本迁移时间线](http://www.zlovezl.cn/static//uploaded/2017/05/2017-05-30-15-51-20_thumb.jpg)



当所有的代码都都迁移到 Python 3 运行环境后：

- 节约了 12% 的整体 CPU 使用率（Django/uwsgi）
- 节约了 30% 的内存使用（celery）

同时，在整个迁移期间，Instagram 的月活用户经历了从 4 亿到 6亿 的巨大增长。产品也发布了评论过滤、直播等非常多新功能。

那么，那几个最开始驱动他们迁移到 Python 3 的目的呢？

- **类型注解**：Instagram 的整个 codebase 里已经有 2% 的代码添加上了类型注解，同时他们还开发了一些工具来辅助开发者添加类型提示
- **asyncio**：他们在单个接口中利用 asynio 平行的去做多件事情，最终降低了 20-30% 的请求延迟。
- **社区**：他们与 Intel 的工程师联合，帮助他们更好的对 CPU 利用率进行调优。同时还开发了很多新的工具，帮助他们进行性能调优



## Instagram 带给我们的启示

Instagram 的演讲视频时间不长，但是内容很丰富，在编写此文前，我完全没有想到最终的文章会这么长。

那么，Instagram 的视频可以给我们哪些启示呢？

- Python + Django 的组合完全可以负载用户数以 10 亿记的服务，如果你正准备开始一个项目，放心使用 Python 吧！
- 完善的单元测试对于复杂项目是非常有必要的。如果没有那『成千上万的单元测试』。很难想象 Instagram 的迁移项目可以成功进行下去。
- 开发者和同事也是你的产品用户，利用好他们。用他们为你的新特性发布前多一道测试。
- 完全基于主分支的开发流程，可以给你更快的迭代速度。前提是拥有完善的单元测试和持续部署流程。
- Python 3 是大势所趋，如果你正准备开始一个新项目，无需迟疑，拥抱 Python 3 吧！

