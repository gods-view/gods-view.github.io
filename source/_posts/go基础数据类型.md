---
title: go基础数据类型
tags:
  - Go
keywords: 博客文章密码
abstract: 密码：first
message: 输入密码，查看文章
abbrlink: 539388a9
date: 2021-02-25 21:33:44
password:
---

## 1. 结构体和指针

与C相同而与Java不同的是，Go语言让程序员决定何时使用指针。举例来说，这种类型定义：

```go
type Point struct { X, Y int }
```

先来定义一个简单的struct类型，名为Point，表示内存中两个相邻的整数。

![image-20201202173141496](https://raw.githubusercontent.com/gods-view/images/master/imageimage-20201202173141496.png)

`Point{10,20}`表示一个已初始化的Point类型。对它进行取地址表示一个指向刚刚分配和初始化的Point类型的指针。前者在内存中是两个词，而后者是一个指向两个词的指针。

结构体的域在内存中是紧挨着排列的。

<!--more-->

```go
type Rect1 struct { Min, Max Point }
type Rect2 struct { Min, Max *Point }
```

![image-20201202173244021](https://raw.githubusercontent.com/gods-view/images/master/imageimage-20201202173244021.png)

Rect1是一个具有两个Point类型属性的结构体，由在一行的两个Point--四个int代表。Rect2是一个具有两个`*Point`类型属性的结构体，由两个*Point表示。

使用过C的程序员可能对`Point`和`*Point`的不同毫不见怪，但用惯Java或Python的程序员们可能就不那么轻松了。Go语言给了程序员基本内存层面的控制，由此提供了诸多能力，如控制给定数据结构集合的总大小、内存分配的次数、内存访问模式以及建立优秀系统的所有要点。



## 2. 字符串

有了前面的准备，我们就可以开始研究更有趣的数据类型了。

![image-20201202173305713](https://raw.githubusercontent.com/gods-view/images/master/imageimage-20201202173305713.png)

（灰色的箭头表示已经实现的但不能直接可见的指针）

字符串在Go语言内存模型中用一个2字长的数据结构表示。它包含一个指向字符串存储数据的指针和一个长度数据。因为string类型是不可变的，对于多字符串共享同一个存储数据是安全的。切分操作`str[i:j]`会得到一个新的2字长结构，一个可能不同的但仍指向同一个字节序列(即上文说的存储数据)的指针和长度数据。这意味着字符串切分可以在不涉及内存分配或复制操作。这使得字符串切分的效率等同于传递下标。

（说句题外话，在Java和其他语言里有一个有名的“疑难杂症”：在你分割字符串并保存时，对于源字符串的引用在内存中仍然保存着完整的原始字符串--即使只有一小部分仍被需要，Go也有这个“毛病”。另一方面，我们努力但又失败了的是，让字符串分割操作变得昂贵--包含一次分配和一次复制。在大多数程序中都避免了这么做。）



## 3. slice

> 切片拥有 **长度** 和 **容量**。
>
> 切片的长度就是它所包含的元素个数。
>
> 切片的容量是从它的第一个元素开始数，到其底层数组元素末尾的个数。
>
> 切片 s 的长度和容量可通过表达式` len(s)` 和 `cap(s)` 来获取。
>

------

一个slice是一个数组某个部分的引用。在内存中，它是一个包含3个域的结构体：指向slice中第一个元素的指针，slice的长度，以及slice的容量。长度是下标操作的上界，如x[i]中i必须小于长度。容量是分割操作的上界，如x[i:j]中j不能大于容量。

![](https://raw.githubusercontent.com/gods-view/images/master/image20201201174947.png)

数组的slice并不会实际复制一份数据，它只是创建一个新的数据结构，包含了另外的一个指针，一个长度和一个容量数据。 如同分割一个字符串，分割数组也不涉及复制操作：它只是新建了一个结构来放置一个不同的指针，长度和容量。在例子中，对[]int{2,3,5,7,11}求值操作会创建一个包含五个值的数组，并设置x的属性来描述这个数组。分割表达式x[1:3]并不分配更多的数据：它只是写了一个新的slice结构的属性来引用相同的存储数据。在例子中，长度为2--只有y[0]和y[1]是有效的索引，但是容量为4--y[0:4]是一个有效的分割表达式。

由于slice是不同于指针的多字长结构，分割操作并不需要分配内存，甚至没有通常被保存在堆中的slice头部。这种表示方法使slice操作和在C中传递指针、长度对一样廉价。Go语言最初使用一个指向以上结构的指针来表示slice，但是这样做意味着每个slice操作都会分配一块新的内存对象。即使使用了快速的分配器，还是给垃圾收集器制造了很多没有必要的工作。移除间接引用及分配操作可以让slice足够廉价，以避免传递显式索引。



## 4. map的实现

Go中的map在底层是用哈希表实现的，你可以在 $GOROOT/src/pkg/runtime/hashmap.goc 找到它的实现。

### 4.1 数据结构

哈希表的数据结构中一些关键的域如下所示：

```go
struct Hmap
{
    uint8   B;    // 可以容纳2^B个项
    uint16  bucketsize;   // 每个桶的大小

    byte    *buckets;     // 2^B个Buckets的数组
    byte    *oldbuckets;  // 前一个buckets，只有当正在扩容时才不为空
};
```

上面给出的结构体只是Hmap的部分的域。需要注意到的是，这里直接使用的是Bucket的数组，而不是Bucket*指针的数组。这意味着，第一个Bucket和后面溢出链的Bucket分配有些不同。第一个Bucket是用的一段连续的内存空间，而后面溢出链的Bucket的空间是使用mallocgc分配的。

这个hash结构使用的是一个可扩展哈希的算法，由hash值mod当前hash表大小决定某一个值属于哪个桶，而hash表大小是2的指数，即上面结构体中的2^B。每次扩容，会增大到上次大小的两倍。结构体中有一个buckets和一个oldbuckets是用来实现增量扩容的。正常情况下直接使用buckets，而oldbuckets为空。如果当前哈希表正在扩容中，则oldbuckets不为空，并且buckets大小是oldbuckets大小的两倍。

具体的Bucket结构如下所示：

```go
struct Bucket
{
    uint8  tophash[BUCKETSIZE]; // hash值的高8位....低位从bucket的array定位到bucket
    Bucket *overflow;           // 溢出桶链表，如果有
    byte   data[1];             // BUCKETSIZE keys followed by BUCKETSIZE values
};
```

其中BUCKETSIZE是用宏定义的8，每个bucket中存放最多8个key/value对, 如果多于8个，那么会申请一个新的bucket，并将它与之前的bucket链起来。

按key的类型采用相应的hash算法得到key的hash值。将hash值的低位当作Hmap结构体中buckets数组的index，找到key所在的bucket。将hash的高8位存储在了bucket的tophash中。**注意，这里高8位不是用来当作key/value在bucket内部的offset的，而是作为一个主键，在查找时对tophash数组的每一项进行顺序匹配的**。先比较hash值高位与bucket的tophash[i]是否相等，如果相等则再比较bucket的第i个的key与所给的key是否相等。如果相等，则返回其对应的value，反之，在overflow buckets中按照上述方法继续寻找。

整个hash的存储如下图所示(临时先采用了XX同学画的图，这个图有点问题)：

![image-20201202173456850](https://raw.githubusercontent.com/gods-view/images/master/imageimage-20201202173456850.png)

图2.2 HMap的存储结构

注意一个细节是Bucket中key/value的放置顺序，是将keys放在一起，values放在一起，为什么不将key和对应的value放在一起呢？如果那么做，存储结构将变成key1/value1/key2/value2… 设想如果是这样的一个map[int64]int8，考虑到字节对齐，会浪费很多存储空间。不得不说通过上述的一个小细节，可以看出Go在设计上的深思熟虑。

### 4.2 增量扩容

大家都知道哈希表表就是以空间换时间，访问速度是直接跟填充因子相关的，所以当哈希表太满之后就需要进行扩容。

如果扩容前的哈希表大小为2^B，扩容之后的大小为2^(B+1)，每次扩容都变为原来大小的两倍，哈希表大小始终为2的指数倍，则有(hash mod 2^B)等价于(hash & (2^B-1))。这样可以简化运算，避免了取余操作。

假设扩容之前容量为X，扩容之后容量为Y，对于某个哈希值hash，一般情况下(hash mod X)不等于(hash mod Y)，所以扩容之后要重新计算每一项在哈希表中的新位置。当hash表扩容之后，需要将那些旧的pair重新哈希到新的table上(源代码中称之为evacuate)， 这个工作并没有在扩容之后一次性完成，而是逐步的完成（在insert和remove时每次搬移1-2个pair），Go语言使用的是增量扩容。

为什么会增量扩容呢？主要是缩短map容器的响应时间。假如我们直接将map用作某个响应实时性要求非常高的web应用存储，如果不采用增量扩容，当map里面存储的元素很多之后，扩容时系统就会卡往，导致较长一段时间内无法响应请求。不过增量扩容本质上还是将总的扩容时间分摊到了每一次哈希操作上面。

扩容会建立一个大小是原来2倍的新的表，将旧的bucket搬到新的表中之后，并不会将旧的bucket从oldbucket中删除，而是加上一个已删除的标记。

正是由于这个工作是逐渐完成的，这样就会导致一部分数据在old table中，一部分在new table中， 所以对于hash table的insert, remove, lookup操作的处理逻辑产生影响。只有当所有的bucket都从旧表移到新表之后，才会将oldbucket释放掉。

扩容的填充因子是多少呢？如果grow的太频繁，会造成空间的利用率很低， 如果很久才grow，会形成很多的overflow buckets，查找的效率也会下降。 这个平衡点如何选取呢(在go中，这个平衡点是有一个宏控制的(#define LOAD 6.5), 它的意思是这样的，如果table中元素的个数大于table中能容纳的元素的个数， 那么就触发一次grow动作。那么这个6.5是怎么得到的呢？原来这个值来源于作者的一个测试程序，遗憾的是没能找到相关的源码，不过作者给出了测试的结果：

```
        LOAD    %overflow  bytes/entry     hitprobe    missprobe
        4.00         2.13        20.77         3.00         4.00
        4.50         4.05        17.30         3.25         4.50
        5.00         6.85        14.77         3.50         5.00
        5.50        10.55        12.94         3.75         5.50
        6.00        15.27        11.67         4.00         6.00
        6.50        20.90        10.79         4.25         6.50
        7.00        27.14        10.15         4.50         7.00
        7.50        34.03         9.73         4.75         7.50
        8.00        41.10         9.40         5.00         8.00

 %overflow   = percentage of buckets which have an overflow bucket
 bytes/entry = overhead bytes used per key/value pair
 hitprobe    = # of entries to check when looking up a present key
 missprobe   = # of entries to check when looking up an absent key
```

可以看出作者取了一个相对适中的值。

### 4.3 查找过程

1. 根据key计算出hash值。
2. 如果存在old table, 首先在old table中查找，如果找到的bucket已经evacuated，转到步骤3。 反之，返回其对应的value。
3. 在new table中查找对应的value。

这里一个细节需要注意一下。不认真看可能会以为低位用于定位bucket在数组的index，那么高位就是用于key/valule在bucket内部的offset。事实上高8位不是用作offset的，而是用于加快key的比较的。

```
do { //对每个桶b
    //依次比较桶内的每一项存放的tophash与所求的hash值高位是否相等
    for(i = 0, k = b->data, v = k + h->keysize * BUCKETSIZE; i < BUCKETSIZE; i++, k += h->keysize, v += h->valuesize) {
        if(b->tophash[i] == top) { 
            k2 = IK(h, k);
            t->key->alg->equal(&eq, t->key->size, key, k2);
            if(eq) { //相等的情况下再去做key比较...
                *keyp = k2;
                return IV(h, v);
            }
        }
    }
    b = b->overflow; //b设置为它的下一下溢出链
} while(b != nil);
```

### 4.4 插入过程分析

1. 根据key算出hash值，进而得出对应的bucket。
2. 如果bucket在old table中，将其重新散列到new table中。
3. 在bucket中，查找空闲的位置，如果已经存在需要插入的key，更新其对应的value。
4. 根据table中元素的个数，判断是否grow table。
5. 如果对应的bucket已经full，重新申请新的bucket作为overbucket。
6. 将key/value pair插入到bucket中。

这里也有几个细节需要注意一下。

在扩容过程中，oldbucket是被冻结的，查找时会在oldbucket中查找，但不会在oldbucket中插入数据。如果在oldbucket是找到了相应的key，做法是将它迁移到新bucket后加入evalucated标记。并且还会额外的迁移另一个pair。

然后就是只要在某个bucket中找到第一个空位，就会将key/value插入到这个位置。也就是位置位于bucket前面的会覆盖后面的(类似于存储系统设计中做删除时的常用的技巧之一，直接用新数据追加方式写，新版本数据覆盖老版本数据)。找到了相同的key或者找到第一个空位就可以结束遍历了。不过这也意味着做删除时必须完全的遍历bucket所有溢出链，将所有的相同key数据都删除。所以目前map的设计是为插入而优化的，删除效率会比插入低一些。

### 4.5 map设计中的性能优化

读完map源代码发现作者还是做了很多设计上的选择的。本人水平有限，谈不上优劣的点评，这里只是拿出来与读者分享。

HMap中是Bucket的数组，而不是Bucket指针的数组。好的方面是可以一次分配较大内存，减少了分配次数，避免多次调用mallocgc。但相应的缺点，其一是可扩展哈希的算法并没有发生作用，扩容时会造成对整个数组的值拷贝(如果实现上用Bucket指针的数组就是指针拷贝了，代价小很多)。其二是首个bucket与后面产生了不一致性。这个会使删除逻辑变得复杂一点。比如删除后面的溢出链可以直接删除，而对于首个bucket，要等到evalucated完毕后，整个oldbucket删除时进行。

没有重用设freelist重用删除的结点。作者把这个加了一个TODO的注释，不过想了一下觉得这个做的意义不大。因为一方面，bucket大小并不一致，重用比较麻烦。另一方面，下层存储已经做过内存池的实现了，所以这里不做重用也会在内存分配那一层被重用的，

bucket直接key/value和间接key/value优化。这个优化做得蛮好的。注意看代码会发现，如果key或value小于128字节，则它们的值是直接使用的bucket作为存储的。否则bucket中存储的是指向实际key/value数据的指针，

bucket存8个key/value对。查找时进行顺序比较。第一次发现高位居然不是用作offset，而是用于加快比较的。定位到bucket之后，居然是一个顺序比较的查找过程。后面仔细想了想，觉得还行。由于bucket只有8个，顺序比较下来也不算过分。仍然是O(1)只不过前面系数大一点点罢了。相当于hash到一个小范围之后，在这个小范围内顺序查找。

插入删除的优化。前面已经提过了，插入只要找到相同的key或者第一个空位，bucket中如果存在一个以上的相同key，前面覆盖后面的(只是如果，实际上不会发生)。而删除就需要遍历完所有bucket溢出链了。这样map的设计就是为插入优化的。考虑到一般的应用场景，这个应该算是很合理的。



## 5. nil类型

什么？nil是一种数据结构么？为什么会讲到它，没搞错吧？没搞错。不仅仅是Go语言中，每门语言中nil都是非常重要的，它代表的是空值的语义。

在不同语言中，表示空这个概念都有细微不同。比如在scheme语言(一种lisp方言)中，nil是true的！而在ruby语言中，一切都是对象，连nil也是一个对象！在C中NULL跟0是等价的。

按照Go语言规范，任何类型在未初始化时都对应一个零值：布尔类型是false，整型是0，字符串是""，而指针，函数，interface，slice，channel和map的零值都是nil。

### 5.1 interface

一个interface在没有进行初始化时，对应的值是nil。也就是说`var v interface{}`，

此时v就是一个nil。在底层存储上，它是一个空指针。与之不同的情况是，interface值为空。比如：

```go
var v *T
var i interface{}
i = v
```

此时i是一个interface，它的值是nil，但它自身不为nil。

Go中的error其实就是一个实现了Error方法的接口：

```go
type error interface {
    Error() string
}
```

因此，我们可以自定义一个error：

```go
type Error struct {
    errCode uint8
}
func (e *Error) Error() string {
        switch e.errCode {
        case 1:
                return "file not found"
        case 2:
                return "time out"
        case 3:
                return "permission denied"
        default:
                return "unknown error"
         }
}
```

如果我们这样使用它：

```go
func checkError(err error) {
    if err != nil {
        panic(err)
    }
}
var e *Error
checkError(e)
```

e是nil的，但是当我们checkError时就会panic。请读者思考一下为什么？

总之，interface跟C语言的指针一样非常灵活，关于空的语义，也跟空指针一样容易困扰新手的，需要注意。

### 5.2 string和slice

string的空值是""，它是不能跟nil比较的。即使是空的string，它的大小也是两个机器字长的。slice也类似，它的空值并不是一个空指针，而是结构体中的指针域为空，空的slice的大小也是三个机器字长的。

### 5.3 channel和map

channel跟string或slice有些不同，它在栈上只是一个指针，实际的数据都是由指针所指向的堆上面。

跟channel相关的操作有：初始化/读/写/关闭。channel未初始化值就是nil，未初始化的channel是不能使用的。下面是一些操作规则：

- 读或者写一个nil的channel的操作会永远阻塞。
- 读一个关闭的channel会立刻返回一个channel元素类型的零值。
- 写一个关闭的channel会导致panic。

map也是指针，实际数据在堆中，未初始化的值是nil。
