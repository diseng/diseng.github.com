---
layout: post
title : 漫谈一致性哈希
category : datastruct
---
{% include JB/setup %}

### 1. 什么是用户体验

说起用户体验，大家一般会聚焦到前端，比如视觉设计、交互设计、界面设计、感情体验等等，但是其实HTTP请求次数、后端接口响应时间、错误提示是否人性化等等也是非常重要的用户体验

### 2. 漫谈

由于技术体验部有用研、设计、前端、后端的同学，本次分享的话题偏向技术性，但是力求每个人都能理解，所以只做漫谈，想要进一步了解技术细节的，可以会后再做交流

### 3. hash

hash是个什么东西，其实大家把它理解成一个数学函数就行，比如f(x)=x+2，就是你输入一个东西，我根据hash算法，来计算出一个值给你，这个过程就叫hash，当然hash还有一些别的要求，比如计算结果一般比输入值要小，结果格式固定等等，我们不需要了解那么细致

计算过程不一样，就是不一样的hash算法，今天我们要讲的是一致性hash算法，为什么要讲这个hash算法，这个要从缓存说起

### 4. 缓存

如果一个网站，你操作一下，网页需要进行10秒的时间才有反应，你觉得这个体验怎么样？

笔者负责的一个应用去年有一天出现了JVM 频繁FullGC导致的宕机，现象非常严重，页面直接502

后面就是仔细理清业务逻辑，然后排查问题，这次不仅解决了宕机问题，因为使用了缓存，发现响应时间有了质的提升

1月份的时候，我又对一些频繁的远程调用、处理比较耗时并且结果基本不变的地方，进行了缓存优化


### 5. 缓存服务器容错性、扩展性

我们使用缓存，非常简单，但是不知道大家有没有考虑过缓存服务器本身的容错性和扩展性

我们先假设有三台缓存服务器

![screenshot](http://img1.tbcdn.cn/L1/461/1/49bb8494df3412d439606c60b893204876b294a6.png)

这个时候我们怎么存放缓存信息？最简单的就是随机存放，但是随机存放有非常明显的问题

一个是同一份数据可能被存在不同的机器上而造成数据冗余，第二个是有可能某数据已经被缓存但是访问却没有命中，因为无法保证对相同key的所有访问都被发送到相同的服务器，为了解决这个问题，我们可以使用简单的hash算法，比如

```
h = Hash(key) % 3
```

这里的hash作用是将输入的key，转换成一个整数，然后对3取余数，所以计算出来的结果只有0，1，2三个数字，并且相同的key每次计算出来的结果肯定是相同的

假设我们有N台机器，现在我们把上面的公式抽象，就是

```
h = Hash(key) % N
```

现在我们解决了随机存放带来的两个问题，看起来似乎不错，但是大家想一下，如果N台机器里面，万一有一台机器X挂了，会出现什么样的情况。

首先挂了的这台机器X无论是存放还是获取操作，都是会失败的，所以一般监控系统检测到有机器挂了，肯定会把这台机器摘除出去，机器数就从N台减到了N-1台，那么X后面的机器就需要序号减1，并且计算公式需要调整成

```
h = Hash(key) % (N-1)
```
![screenshot](http://img4.tbcdn.cn/L1/461/1/e5f1097fe19b7818b3b2f1504fed2fe6a8026b39.png)

我们再来假设另外一种情况，现在有N台机器提供服务，但是已经支持不下去了，所以新增了一台机器，那么计算公式需要调整成

```
h = Hash(key) % (N+1)
```

![screenshot](http://img2.tbcdn.cn/L1/461/1/07b7934b291c8ddd778bf0f31cc4c05a384bedcf.png)

大家发现上面有什么问题没有，就是每次有机器挂了，或者新增机器的时候，都需要调整计算公式，这会导致大量的缓存命中不了，我们把这个叫做容错性和扩展性不强，怎么解决这个问题？

我们来看看一致性hash是怎么做的

### 6. 一致性hash

一个设计良好的分布式哈希方案应该具有良好的单调性，即服务节点的增减不会造成大量哈希重定位。一致性哈希算法就是这样一种哈希方案。最早是1997年由麻省理工大学提出的。

简单来说，一致性哈希将整个哈希值空间组织成一个虚拟的圆环，如假设某哈希函数H的值空间为0 - 232-1（即哈希值是一个32位无符号整形），整个哈希空间环如下：

![screenshot](http://img1.tbcdn.cn/L1/461/1/35dae12424640d4f8e9ff89514dae0f8d2503339.png)

整个空间按顺时针方向组织。0和232-1在零点中方向重合。

下一步将各个服务器使用H进行一个哈希，具体可以选择服务器的ip或主机名作为关键字进行哈希，这样每台机器就能确定其在哈希环上的位置，这里假设将上文中三台服务器使用ip地址哈希后在环空间的位置如下：

![screenshot](http://img2.tbcdn.cn/L1/461/1/b3aec8854b1d445256d7eaf36cf65dfbb157fd1e.png)

接下来使用如下算法定位数据访问到相应服务器：将数据key使用相同的函数H计算出哈希值h，通根据h确定此数据在环上的位置，从此位置沿环顺时针“行走”，第一台遇到的服务器就是其应该定位到的服务器。

例如我们有A、B、C、D四个数据对象，经过哈希计算后，在环空间上的位置如下：

![screenshot](http://img2.tbcdn.cn/L1/461/1/2a65577f7f95de04df2586231598dc6da46385f0.png)

根据一致性哈希算法，数据A会被定为到Server 1上，D被定为到Server 3上，而B、C分别被定为到Server 2上。

### 7.一致性hash的容错性与可扩展性

下面分析一致性哈希算法的容错性和可扩展性。现假设Server 3宕机了：

![screenshot](http://img4.tbcdn.cn/L1/461/1/19f747ca07f45c684178545e0aac40a6f3f8287e.png)

可以看到此时A、C、B不会受到影响，只有D节点被重定位到Server 2。一般的，在一致性哈希算法中，如果一台服务器不可用，则受影响的数据仅仅是此服务器到其环空间中前一台服务器（即顺着逆时针方向行走遇到的第一台服务器）之间数据，其它不会受到影响。

下面考虑另外一种情况，如果我们在系统中增加一台服务器Memcached Server 4：

![screenshot](http://img4.tbcdn.cn/L1/461/1/dcd96cc46aba1f6c289e1e6ed5e08438004d3976.png)

此时A、D、C不受影响，只有B需要重定位到新的Server 4。一般的，在一致性哈希算法中，如果增加一台服务器，则受影响的数据仅仅是新服务器到其环空间中前一台服务器（即顺着逆时针方向行走遇到的第一台服务器）之间数据，其它不会受到影响。

综上所述，一致性哈希算法对于节点的增减都只需重定位环空间中的一小部分数据，具有较好的容错性和可扩展性。

### 8. 一致性hash的虚拟节点

一致性哈希算法在服务节点太少时，容易因为节点分部不均匀而造成数据倾斜问题。例如我们的系统中有两台服务器，其环分布如下：

![screenshot](http://img1.tbcdn.cn/L1/461/1/fe5c17d1a5c4fdfe15baebc96412c50cdec6d9b4.png)

此时必然造成大量数据集中到Server 1上，而只有极少量会定位到Server 2上。为了解决这种数据倾斜问题，一致性哈希算法引入了虚拟节点机制，即对每一个服务节点计算多个哈希，每个计算结果位置都放置一个此服务节点，称为虚拟节点。具体做法可以在服务器ip或主机名的后面增加编号来实现。例如上面的情况，我们决定为每台服务器计算三个虚拟节点，于是可以分别计算“Memcached Server 1#1”、“Memcached Server 1#2”、“Memcached Server 1#3”、“Memcached Server 2#1”、“Memcached Server 2#2”、“Memcached Server 2#3”的哈希值，于是形成六个虚拟节点：

![screenshot](http://img4.tbcdn.cn/L1/461/1/01ca5792121bd0327f2232866345f20d02429c3f.png)

同时数据定位算法不变，只是多了一步虚拟节点到实际节点的映射，例如定位到“Memcached Server 1#1”、“Memcached Server 1#2”、“Memcached Server 1#3”三个虚拟节点的数据均定位到Server 1上。这样就解决了服务节点少时数据倾斜的问题。在实际应用中，通常将虚拟节点数设置为32甚至更大，因此即使很少的服务节点也能做到相对均匀的数据分布。

### 9. 其他

本文参考文章：[一致性哈希算法及其在分布式系统中的应用](http://blog.codinglabs.org/articles/consistent-hashing.html)
维基百科：[散列函数](https://zh.wikipedia.org/wiki/%E6%95%A3%E5%88%97%E5%87%BD%E6%95%B8)