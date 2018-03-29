---
title: "xUID 三种唯一ID生成"
subtitle: GUID-UUID-NUID
date: 2018-03-27T16:49:44+08:00
tags: [ "Go", "Github", "GUID", "Go Libs" ]
description: 记录一些值得学习的Go开源项目
categories: [ "Go" ]
draft: false
isCJKLanguage: true
---

开发项目中可能经常会使用到唯一标识符（`Unique Identifier`），特别是设计一些并发并行的应用系统，之前在Github上看到了一些生成算法库，在此做些学习记录，以备或需。
<br />
### UUID

------

第一种`UUID`，这个就不用多说了，很多语言标准库里面都提供了生成函数，Go的标准库里面没有，但是万能的开源社区提供了多个项目，用来实现符合`RFC4122`规范的`UUID`。

> 我看了两个项目：
>
> - https://github.com/nu7hatch/gouuid
>
> - https://github.com/satori/go.uuid
>

<!--more-->
前面那个提供了`NewV3`、`NewV4`、`NewV5`三个版本的`UUID`生成函数；后面那个提供了五种版本的`UUID`生成函数以及丰富的检验、序列化/反序列化、编码解码等函数，更有实用价值。关于`UUID`的`五种版本`规范，可以百度谷歌，这里提供 [SegmentFault上一个回答的链接](https://segmentfault.com/q/1010000010862121) 。源码大致浏览了一下，根据不同版本选择一些参与计算的要素，例如：`时间`、`MAC地址`、`名字空间`、`MD5`、`SHA1`等，对RFC规范不熟悉，具体算法实现不仔细看了，需要的时候用就可以了。

<br />
### snowflake
------

这个算法是`Twitter`提出来并开源的，具体可以参见：https://github.com/twitter/snowflake/tree/snowflake-2010 。Github上Go语言实现这个算法的项目也很多，我选择这个项目学习了一下：

> - https://github.com/zheng-ji/goSnowFlake

网上对这个算法的分析文章也很多，贴一篇不错的：[[理解分布式id生成算法SnowFlake](https://segmentfault.com/a/1190000011282426)](https://segmentfault.com/a/1190000011282426) 。

这个算法生成的`ID`是一个`64位整数`，原理是某个时间点（2^41^-1毫秒）、某台主机（`1024`个节点），按序列号（最大`4096`个）生成一个数。比如如果用作交易流水号，也就是说一毫秒内某台节点受理并发交易不超过`4096`笔，就不会有重复的可能，还是很强劲的。

简单贴下实现的部分源码：

```go
const (
	CEpoch         = 1474802888000
	CWorkerIdBits  = 10 // Num of WorkerId Bits
	CSenquenceBits = 12 // Num of Sequence Bits

	CWorkerIdShift  = 12
	CTimeStampShift = 22

	CSequenceMask = 0xfff // equal as getSequenceMask()
	CMaxWorker    = 0x3ff // equal as getMaxWorkerId()
)
```

```go
// +---------------+----------------+----------------+
// |timestamp(ms)42  | worker id(10) | sequence(12)	 |
// +---------------+----------------+----------------+
// NewId Func: Generate next id
func (iw *IdWorker) NextId() (ts int64, err error) {
	iw.lock.Lock()
	defer iw.lock.Unlock()
	ts = iw.timeGen()
	if ts == iw.lastTimeStamp {
		iw.sequence = (iw.sequence + 1) & CSequenceMask
		if iw.sequence == 0 {
			ts = iw.timeReGen(ts)
		}
	} else {
		iw.sequence = 0
	}

	if ts < iw.lastTimeStamp {
		err = errors.New("Clock moved backwards, Refuse gen id")
		return 0, err
	}
	iw.lastTimeStamp = ts
	ts = (ts-CEpoch)<<CTimeStampShift | iw.workerId<<CWorkerIdShift | iw.sequence
	return ts, nil
}
```

<br />
### NUID
------

最后，看一个我比较喜欢的`UID`生成算法，它是最近才加入`CNCF`组织的`NATs`系列项目中使用到的一种唯一标识符算法库，生成速度极快，重复可能极其低。

> - https://github.com/nats-io/nuid

这个被命名为`NUID`的算法生成的是一个22字节的数字大小写字母字符串，因此生成范围可能性是62^22^，生成速度我老Mac笔记本大约是`60ns~80ns`左右一个。`2000万`只需一点几秒，真是极速爽啊。

看一下算法规则和源码：

```go
const (
	digits   = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
	base     = 62
	preLen   = 12
	seqLen   = 10
	maxSeq   = int64(839299365868340224) // base^seqLen == 62^10
	minInc   = int64(33)
	maxInc   = int64(333)
	totalLen = preLen + seqLen
)
```

```go
func (n *NUID) Next() string {
	// Increment and capture.
	n.seq += n.inc
	if n.seq >= maxSeq {
		n.RandomizePrefix()
		n.resetSequential()
	}
	seq := n.seq

	// Copy prefix
	var b [totalLen]byte
	bs := b[:preLen]
	copy(bs, n.pre)

	// copy in the seq in base36.
	for i, l := len(b), seq; i > preLen; l /= base {
		i -= 1
		b[i] = digits[l%base]
	}
	return string(b[:])
}
```

算法是`12字节随机前缀字符串+10字节序列号字符串`，两个字符串都是先由随机数或者随机数+随机递增数产生后，再逐数字位随机读取再转换成数字大小写字母。算法很精炼，可以仔细阅读源码研究。

另外，项目源码中还有生成`NUID`的测试和性能测试程序，还有唯一性检测程序，可以测试和检验。