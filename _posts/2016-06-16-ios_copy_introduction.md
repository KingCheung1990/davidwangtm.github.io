---
layout:     post
title:      "ios 进阶篇"
subtitle:   "iOS 集合的深复制与浅复制"
date:       2016-06-16 09:30:00
author:     "DavidWang"
header-img: "img/post-bg-rwd.jpg"
catalog: true
tags:
    - ios
---  

### 概念

对象拷贝有两种方式：浅复制和深复制。顾名思义，浅复制，并不拷贝对象本身，仅仅是拷贝指向对象的指针；深复制是直接拷贝整个对象内存到另一块内存中。

一图以蔽之

![URL](/img/in-post/ios_introduction/ios_image_note50592_1.png)

再简单些说：**浅复制就是指针拷贝；深复制就是内容拷贝**。

### 集合的浅复制 (shallow copy)

集合的浅复制有非常多种方法。当你进行浅复制时，会向原始的集合发送retain消息，引用计数加1，同时指针被拷贝到新的集合。

现在让我们看一些浅复制的例子：

```
NSArray *shallowCopyArray = [someArray copyWithZone:nil];
NSSet *shallowCopySet = [NSSet mutableCopyWithZone:nil];
NSDictionary *shallowCopyDict = [[NSDictionary alloc] initWithDictionary:someDictionary copyItems:NO];
```

### 集合的深复制 (deep copy)

集合的深复制有两种方法。可以用 initWithArray:copyItems: 将第二个参数设置为YES即可深复制，如

```
NSDictionary shallowCopyDict = [[NSDictionary alloc] initWithDictionary:someDictionary copyItems:YES];
```

如果你用这种方法深复制，集合里的每个对象都会收到 copyWithZone: 消息。如果集合里的对象遵循 NSCopying 协议，那么对象就会被深复制到新的集合。如果对象没有遵循 NSCopying 协议，而尝试用这种方法进行深复制，会在运行时出错。copyWithZone: 这种拷贝方式只能够提供一层内存拷贝(one-level-deep copy)，而非真正的深复制。

第二个方法是将集合进行归档(archive)，然后解档(unarchive)，如：

```
NSArray *trueDeepCopyArray = [NSKeyedUnarchiver unarchiveObjectWithData:[NSKeyedArchiver archivedDataWithRootObject:oldArray]];
```

### 集合的单层深复制 (one-level-deep copy)

看到这里，有同学会问：如果在多层数组中，对第一层进行内容拷贝，其它层进行指针拷贝，这种情况是属于深复制，还是浅复制？对此，[苹果官网文档](https://developer.apple.com/library/mac/documentation/cocoa/conceptual/Collections/Articles/Copying.html#//apple_ref/doc/uid/TP40010162-SW3)有这样一句话描述

```
This kind of copy is only capable of producing a one-level-deep copy. If you only need a one-level-deep copy...

If you need a true deep copy, such as when you have an array of arrays...
```

从文中可以看出，苹果认为这种复制不是真正的深复制，而是将其称为**单层深复制(one-level-deep copy)**。因此，网上有人对浅复制、深复制、单层深复制做了概念区分。

- 浅复制(shallow copy)：在浅复制操作时，对于被复制对象的每一层都是指针复制。
- 深复制(one-level-deep copy)：在深复制操作时，对于被复制对象，至少有一层是深复制。
- 完全复制(real-deep copy)：在完全复制操作时，对于被复制对象的每一层都是对象复制。

当然，这些都是概念性的东西，没有必要纠结于此。只要知道进行拷贝操作时，被拷贝的是指针还是内容即可。

### 系统对象的copy与mutableCopy方法

不管是集合类对象，还是非集合类对象，接收到copy和mutableCopy消息时，都遵循以下准则：

- copy返回imutable对象；所以，如果对copy返回值使用mutable对象接口就会crash；
- mutableCopy返回mutable对象；

下面将针对非集合类对象和集合类对象的copy和mutableCopy方法进行具体的阐述

#### 1、非集合类对象的copy与mutableCopy

系统非集合类对象指的是 NSString, NSNumber ... 之类的对象。下面先看个非集合类immutable对象拷贝的例子

```
NSString *string = @"origin";
NSString *stringCopy = [string copy];
NSMutableString *stringMCopy = [string mutableCopy];
```
通过查看内存，可以看到 stringCopy 和 string 的地址是一样，进行了指针拷贝；而 stringMCopy 的地址和 string 不一样，进行了内容拷贝；

再看mutable对象拷贝例子

```
NSMutableString *string = [NSMutableString stringWithString: @"origin"];
//copy
NSString *stringCopy = [string copy];
NSMutableString *mStringCopy = [string copy];
NSMutableString *stringMCopy = [string mutableCopy];
//change value
[mStringCopy appendString:@"mm"]; //crash
[string appendString:@" origion!"];
[stringMCopy appendString:@"!!"];
```

运行以上代码，会在第7行crash，原因就是 copy 返回的对象是 immutable 对象。注释第7行后再运行，查看内存，发现 string、stringCopy、mStringCopy、stringMCopy 四个对象的内存地址都不一样，说明此时都是做内容拷贝。

综上两个例子，我们可以得出结论：

```
在非集合类对象中：对immutable对象进行copy操作，是指针复制，mutableCopy操作时内容复制；对mutable对象进行copy和mutableCopy都是内容复制。用代码简单表示如下：

[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //深复制
[mutableObject copy] //深复制
[mutableObject mutableCopy] //深复制
```

#### 2、集合类对象的copy与mutableCopy

集合类对象是指NSArray、NSDictionary、NSSet ... 之类的对象。下面先看集合类immutable对象使用copy和mutableCopy的一个例子：

```
NSArray *array = @[@[@"a", @"b"], @[@"c", @"d"];
NSArray *copyArray = [array copy];
NSMutableArray *mCopyArray = [array mutableCopy];
```
查看内容，可以看到copyArray和array的地址是一样的，而mCopyArray和array的地址是不同的。说明copy操作进行了指针拷贝，mutableCopy进行了内容拷贝。但需要强调的是：此处的内容拷贝，仅仅是拷贝array这个对象，array集合内部的元素仍然是指针拷贝。这和上面的非集合immutable对象的拷贝还是挺相似的，那么mutable对象的拷贝会不会类似呢？我们继续往下，看mutable对象拷贝的例子：

```
NSMutableArray *array = [NSMutableArray arrayWithObjects:[NSMutableString stringWithString:@"a"],@"b",@"c",nil];
NSArray *copyArray = [array copy];
NSMutableArray *mCopyArray = [array mutableCopy];
```
查看内存，如我们所料，copyArray、mCopyArray和array的内存地址都不一样，说明copyArray、mCopyArray都对array进行了内容拷贝。同样，我们可以得出结论：

```
在集合类对象中，对immutable对象进行copy，是指针复制，mutableCopy是内容复制；对mutable对象进行copy和mutableCopy都是内容复制。但是：集合对象的内容复制仅限于对象本身，对象元素仍然是指针复制。用代码简单表示如下：

[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //单层深复制
[mutableObject copy] //单层深复制
[mutableObject mutableCopy] //单层深复制
```
这个代码结论和非集合类的非常相似。

这时候，是不是有人要问了，如果要对集合对象复制元素怎么办？有这疑问的同学不妨回头看看**集合的深复制**。

好了，深复制与浅复制就讲到这里。

最后说个题外的东西，在搜集资料的过程中，发现一个有可能犯错的点

```
NSString *str = @"string";
str = @"newString";
```

上面这段代码，在执行第二行代码后，内存地址发生了变化。乍一看，有点意外。按照 C 语言的经验，初始化一个字符串之后，字符串的首地址就被确定下来，不管之后如何修改字符串内容，这个地址都不会改变。但此处第二行并不是对 str 指向的内存地址重新赋值，因为赋值操作符左边的 str 是一个指针，也就是说此处修改的是内存地址。

所以第二行应该这样理解：将@"newStirng"当做一个新的对象，将这段对象的内存地址赋值给str。

我有如下的两个方法查看内存地址

- `p str`会打印对象本身的内存地址和对象内容

```
(lldb) p str
(NSString *) $0 = 0x000000010c913680 @"a"
```
- `po &str` 则打印的是引用对象的指针所在的地址

```
(lldb) po &str
0x00007fff532fb6c0
```
