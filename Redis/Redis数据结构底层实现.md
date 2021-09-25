redis有五种基本数据结构：字符串、hash、set、zset、list。但是你知道构成这五种结构的底层数据结构是怎样的吗？ 今天我们来花费五分钟的时间了解一下。 (目前redis版本为3.0.6)

# 动态字符串SDS

SDS是"simple dynamic string"的缩写。 redis中所有场景中出现的字符串，基本都是由SDS来实现的

- 所有非数字的key。例如`set msg "hello world"` 中的key msg.
- 字符串数据类型的值。例如`` set msg "hello world"中的msg的值"hello wolrd"
- 非字符串数据类型中的“字符串值”。例如`RPUSH fruits "apple" "banana" "cherry"`中的"apple" "banana" "cherry"

### SDS长这样：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2439a5def291~tplv-t2oaga2asx-watermark.awebp)



free:还剩多少空间 len:字符串长度 buf:存放的字符数组

### 空间预分配

为减少修改字符串带来的内存重分配次数，sds采用了“一次管够”的策略：

- 若修改之后sds长度小于1MB,则多分配现有len长度的空间
- 若修改之后sds长度大于等于1MB，则扩充除了满足修改之后的长度外，额外多1MB空间



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec24406eb1dcd1~tplv-t2oaga2asx-watermark.awebp)



### 惰性空间释放

为避免缩短字符串时候的内存重分配操作，sds在数据减少时，并不立刻释放空间。



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec24437fc8d8d7~tplv-t2oaga2asx-watermark.awebp)



# int

就是redis中存放的各种数字 包括一下这种，故意加引号“”的



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2445c323a9a8~tplv-t2oaga2asx-watermark.awebp)



# 双向链表

长这样：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2445c3896df0~tplv-t2oaga2asx-watermark.awebp)



分两部分，一部分是“统筹部分”：橘黄色，一部分是“具体实施方“：蓝色。

主体”统筹部分“：

- `head`指向具体双向链表的头
- `tail`指向具体双向链表的尾
- `len`双向链表的长度

具体"实施方"：一目了然的双向链表结构，有前驱`pre`有后继`next`

由`list`和`listNode`两个数据结构构成。

# ziplist

压缩列表。 redis的列表键和哈希键的底层实现之一。此数据结构是为了节约内存而开发的。和各种语言的数组类似，它是由连续的内存块组成的，这样一来，由于内存是连续的，就减少了很多内存碎片和指针的内存占用，进而节约了内存。



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2445c3937e4d~tplv-t2oaga2asx-watermark.awebp)



然后文中的`entry`的结构是这样的：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2447c52aa56a~tplv-t2oaga2asx-watermark.awebp)



### 元素的遍历

先找到列表尾部元素：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2447c53c3024~tplv-t2oaga2asx-watermark.awebp)



然后再根据ziplist节点元素中的`previous_entry_length`属性，来逐个遍历:



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2449a7dc6360~tplv-t2oaga2asx-watermark.awebp)



### 连锁更新

再次看看`entry`元素的结构，有一个`previous_entry_length`字段，他的长度要么都是1个字节，要么都是5个字节：

- 前一节点的长度小于254字节，则`previous_entry_length`长度为1字节
- 前一节点的长度大于254字节，则`previous_entry_length`长度为5字节

假设现在存在一组压缩列表，长度都在250字节至253字节之间，突然新增一新节点`new`， 长度大于等于254字节，会出现：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2449a7fb8404~tplv-t2oaga2asx-watermark.awebp)



程序需要不断的对压缩列表进行空间重分配工作，直到结束。

除了增加操作，删除操作也有可能带来“连锁更新”。 请看下图，ziplist中所有entry节点的长度都在250字节至253字节之间，big节点长度大于254字节，small节点小于254字节。



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2449a8205b33~tplv-t2oaga2asx-watermark.awebp)



# 哈希表

哈希表略微有点复杂。哈希表的制作方法一般有两种，一种是：`开放寻址法`，一种是`拉链法`。redis的哈希表的制作使用的是`拉链法`。

整体结构如下图：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec244b573b9c7b~tplv-t2oaga2asx-watermark.awebp)



也是分为两部分：左边橘黄色部分和右边蓝色部分，同样，也是”统筹“和”实施“的关系。 具体哈希表的实现，都是在蓝色部分实现的。 先来看看蓝色部分：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec244b575414d9~tplv-t2oaga2asx-watermark.awebp)



这也分为左右两边“统筹”和“实施”的两部分。

右边部分很容易理解：就是通常拉链表实现的哈希表的样式；数组就是bucket，一般不同的key首先会定位到不同的bucket，若key重复，就用链表把冲突的key串起来。

### 新建key的过程：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec244b575509bb~tplv-t2oaga2asx-watermark.awebp)



### 假如重复了:



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec244dd6e6a6ba~tplv-t2oaga2asx-watermark.awebp)



## rehash

再来看看哈希表总体图中左边橘黄色的“统筹”部分，其中有两个关键的属性：`ht`和`rehashidx`。 `ht`是一个数组，有且只有俩元素ht[0]和ht[1];其中，ht[0]存放的是redis中使用的哈希表，而ht[1]和rehashidx和哈希表的`rehash`有关。

`rehash`指的是重新计算键的哈希值和索引值，然后将键值对重排的过程。

`加载因子（load factor） = ht[0].used / ht[0].size`。

### 扩容和收缩标准

扩容：

- 没有执行BGSAVE和BGREWRITEAOF指令的情况下，哈希表的`加载因子`大于等于1。
- 正在执行BGSAVE和BGREWRITEAOF指令的情况下，哈希表的`加载因子`大于等于5。

收缩:

- `加载因子`小于0.1时，程序自动开始对哈希表进行收缩操作。

### 扩容和收缩的数量

扩容：

- 第一个大于等于`ht[0].used * 2`的`2^n`(2的n次方幂)。

收缩：

- 第一个大于等于`ht[0].used`的`2^n`(2的n次方幂)。

**(以下部分属于细节分析，可以跳过直接看扩容步骤)** 对于收缩，我当时陷入了疑虑：收缩标准是`加载因子`小于0.1的时候，也就是说假如哈希表中有4个元素的话，哈希表的长度只要大于40，就会进行收缩，假如有一个长度大于40，但是存在的元素为4即(`ht[0].used`为4)的哈希表，进行收缩，那收缩后的值为多少？

我想了一下：按照前文所讲的内容，应该是4。 但是，假如是4，存在和收缩后的长度相等，是不是又该扩容？ 翻开源码看看：

收缩具体函数:

```
int dictResize(dict *d)     //缩小字典d
{
    int minimal;

    //如果dict_can_resize被设置成0，表示不能进行rehash，或正在进行rehash，返回出错标志DICT_ERR
    if (!dict_can_resize || dictIsRehashing(d)) return DICT_ERR;

    minimal = d->ht[0].used;            //获得已经有的节点数量作为最小限度minimal
    if (minimal < DICT_HT_INITIAL_SIZE)//但是minimal不能小于最低值DICT_HT_INITIAL_SIZE（4）
        minimal = DICT_HT_INITIAL_SIZE;
    return dictExpand(d, minimal);      //用minimal调整字典d的大小
} 
复制代码
int dictExpand(dict *d, unsigned long size)     //根据size调整或创建字典d的哈希表
{
    dictht n; 
    unsigned long realsize = _dictNextPower(size);  //获得一个最接近2^n的realsize

    if (dictIsRehashing(d) || d->ht[0].used > size) //正在rehash或size不够大返回出错标志
        return DICT_ERR;

    if (realsize == d->ht[0].size) return DICT_ERR; //如果新的realsize和原本的size一样则返回出错标志
    /* Allocate the new hash table and initialize all pointers to NULL */
    //初始化新的哈希表的成员
    n.size = realsize;
    n.sizemask = realsize-1;
    n.table = zcalloc(realsize*sizeof(dictEntry*));
    n.used = 0;

    /* Is this the first initialization? If so it's not really a rehashing
     * we just set the first hash table so that it can accept keys. */
    if (d->ht[0].table == NULL) {   //如果ht[0]哈希表为空，则将新的哈希表n设置为ht[0]
        d->ht[0] = n;
        return DICT_OK;
    }

    d->ht[1] = n;           //如果ht[0]非空，则需要rehash
    d->rehashidx = 0;       //设置rehash标志位为0，开始渐进式rehash（incremental rehashing）
    return DICT_OK;
} 
复制代码
static unsigned long _dictNextPower(unsigned long size)
{
    unsigned long i = DICT_HT_INITIAL_SIZE; //DICT_HT_INITIAL_SIZE 为 4

    if (size >= LONG_MAX) return LONG_MAX + 1LU;
    while(1) {
        if (i >= size)
            return i;
        i *= 2;
    }
}

复制代码
```

由代码我们可以看到，假如收缩后长度为4，不仅不会收缩，甚至还会报错。(😝)

我们回过头来再看看设定：题目可能成立吗？ 哈希表的扩容都是2倍增长的，最小是4， 4 ===》 8 ====》 16 =====》 32 ======》 64 ====》 128

也就是说：不存在长度为 40多的情况，只能是64。但是如果是64的话，64 X 0.1（收缩界限）= 6.4 ，也就是说在减少到6的时候，哈希表就会收缩，会缩小到多少呢？是8。此时，再继续减少到4，也不会再收缩了。所以，根本不存在一个长度大于40，但是存在的元素为4的哈希表的。

### 扩容步骤



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec244dd77c5ff4~tplv-t2oaga2asx-watermark.awebp)



### 收缩步骤



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2451310cf736~tplv-t2oaga2asx-watermark.awebp)



### 渐进式refresh

在"扩容步骤"和"收缩步骤" 两幅动图中每幅图的第四步骤“将ht[0]中的数据利用哈希函数重新计算，rehash到ht[1]”，并不是一步完成的，而是分成N多步，循序渐进的完成的。 因为hash中有可能存放几千万甚至上亿个key，毕竟Redis中每个hash中可以存`2^32 - 1` 键值对（40多亿），假如一次性将这些键值rehash的话，可能会导致服务器在一段时间内停止服务，毕竟哈希函数就得计算一阵子呢((#^.^#))。

哈希表的refresh是分多次、渐进式进行的。

渐进式refresh和下图中左边橘黄色的“统筹”部分中的`rehashidx`密切相关：

- rehashidx 的数值就是现在rehash的元素位置
- rehashidx 等于 -1 的时候说明没有在进行refresh



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec245131189a83~tplv-t2oaga2asx-watermark.awebp)



甚至在进行期间，每次对哈希表的增删改查操作，除了正常执行之外，还会顺带将ht[0]哈希表相关键值对rehash到ht[1]。

以扩容步骤为例：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2452ead60310~tplv-t2oaga2asx-watermark.awebp)



# intset

整数集合是集合键的底层实现方式之一。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2451312ea78e~tplv-t2oaga2asx-watermark.awebp)



# 跳表

跳表这种数据结构长这样：

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec24513168d425~tplv-t2oaga2asx-watermark.awebp)



redis中把跳表抽象成如下所示：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec245131774efc~tplv-t2oaga2asx-watermark.awebp)



看这个图，左边“统筹”，右边实现。 统筹部分有以下几点说明：

- header: 跳表表头
- tail:跳表表尾
- level:层数最大的那个节点的层数
- length：跳表的长度

实现部分有以下几点说明：

- 表头：是链表的哨兵节点，不记录主体数据。
- 是个双向链表
- 分值是有顺序的
- o1、o2、o3是节点所保存的成员，是一个指针，可以指向一个SDS值。
- 层级高度最高是32。没每次创建一个新的节点的时候，程序都会随机生成一个介于1和32之间的值作为level数组的大小，这个大小就是“高度”

# redis五种数据结构的实现

### redis对象

redis中并没有直接使用以上所说的各种数据结构来实现键值数据库，而是基于一种对象，对象底层再间接的引用上文所说的具体的数据结构。

结构如下图：



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2452eb34831c~tplv-t2oaga2asx-watermark.awebp)



### 字符串



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2452eb270d5b~tplv-t2oaga2asx-watermark.awebp)



其中：embstr和raw都是由SDS动态字符串构成的。唯一区别是：raw是分配内存的时候，redisobject和 sds 各分配一块内存，而embstr是redisobject和在一块儿内存中。

### 列表



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2452eb5fe428~tplv-t2oaga2asx-watermark.awebp)



### hash



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2456cccd3e53~tplv-t2oaga2asx-watermark.awebp)



### set



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec2472de84bee9~tplv-t2oaga2asx-watermark.awebp)



### zset



![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/2/16ec24738078de0e~tplv-t2oaga2asx-watermark.awebp)

> 
> https://juejin.cn/post/6844904008591605767

