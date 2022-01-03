---
title: "ARTS27"
date: 2022-01-03T23:35:10+08:00
---

## Algorithm
#### [剑指 Offer 40\. 最小的k个数](https://leetcode-cn.com/problems/zui-xiao-de-kge-shu-lcof/)
很容易想到使用最小堆来解决这个问题，Go里面的最小堆需要实现container/heap的Interface。
```
import "container/heap"

type IntHeap []int

func (h *IntHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
	result := (*h)[len(*h)-1]
	*h = (*h)[0 : len(*h)-1]
	return result
}

func (h IntHeap) Len() int {
	return len(h)
}

func (h IntHeap) Less(i, j int) bool {
	return h[i] < h[j]
}

func (h IntHeap) Swap(i, j int) {
	h[i], h[j] = h[j], h[i]
}

func getLeastNumbers(arr []int, k int) []int {
	result := make([]int, 0, k)
	initArray := &IntHeap{}
	for _, ele := range arr {
		ele := ele
		heap.Push(initArray, ele)
		// heap.Init(initArray)
		if initArray.Len() > len(arr)-k {
			temp := heap.Pop(initArray).(int)
			result = append(result, temp)
		}
	}
	return result
}
```
## Review
[Unique Id generation in distributed systems](https://medium.com/nerd-for-tech/unique-id-generation-in-distributed-systems-6f7aaa39c9af)
文章首先科普了下啥是分布式ID生成器，然后介绍了Twitter snowflake的实现原理，最后自己手动实现了一个简易版ID生成器（原理完全按照Twitter snowflake来实现）
1. 如果对Mysql进行分库的话，如果直接使用MySql的自增ID来当作主键的话，不同数据库分片的主键就会有冲突。这时候就需要一个分布式ID生成器服务来提供unique ID。
1. Twitter snowflake的原理与构成：
  - Epoch timestamp in millisecond — 41 bits (gives us 69 years with respect to any custom epoch)
  - Configured machine/node/shard Id — 10 bits (gives us up to a total of 2^10 i.e 1024 Ids)
  - Sequence number — 12 bits (A local counter per machine that sets to zero after every 4096 values)
  - The extra 1 reserved bit at the beginning is set as 0 to make the overall number as positive.

3. 自己实现一个简易版的分布式ID生成器。关键方法：
  - nextId方法。这个方法需要加锁，主要工作就是拼接时间戳、node ID、序列号生成ID。
  - createNodeId。通过服务器的mac生成node ID。
  - 分布式ID生成器在单个实例生需要是单例的，不然序列号和node ID就会一致，最终导致可能生成的分布式ID会冲突。
## TIP
这周工作过程用到了go vendor，了解到了几个基本的概念和用法。
1. vendor模式下，会优先使用去当前仓库的vendor目录下寻找依赖的SDK。如果找不到再去GoPath路径的src目录下寻找依赖。
1. govendor get xxx会安装并将依赖的SDK源码copy到当前仓库的vendor目录下，这时候vendor.json会新增这个依赖的相关信息。
1. govendor add会将已经安装的依赖copy到当前仓库的vendor目录下，这时候vendor.json会新增这个依赖的相关信息。相比govendor get，只会执行copy的动作。
1. 在go mod仓库下，使用go mod vendor会在root目录下生成vendor目录，并将go.mod的所有依赖copy到vendor目录下。
## Share
继续学习“操作系统32讲”
### L3 操作系统启动
#### 启动过程
1. 引导扇区的bootsect.s会分段将磁盘上的操作系统代码读取到内存中。先读取setup模块，然后读取system模块的代码，然后执行setup和system模块。
2. setup模块是一段汇编代码，主要做的工作就是将操作系统基础信息建立起来。会做以下动作：
  - 读取光标位置
  - 读取扩展内存数
  - 读取显卡参数
  - 读取根设备号
  - 将操作系统system模块挪到0x0000-0x7c00 和后续应用层代码的内存段区分开
  - 通过执行mov ax,#0x0001 mov cr0,ax来将当前系统置为保护模式（32位机模式）。在这个阶段之前，系统的寻址方式是cs:ip模式（cs<<4+ip，cs指的是段寄存器，ip指的是ip寄存器，都是16位的寄存器。能访问的内存地址是2的20次方，也就是1M的内存），这个模式是16位机。
  - 然后通过jmpi 0,8 跳转到system模块所在的内存地址。
3. 执行system模块代码
  - 执行head.s。head.s是一段汇编代码，是system的第一个文件。再次初始化gdt表。
  - 执行main.c。main.c是一段c代码。main函数做了一系列的init函数，包括内存、中断、设备、时钟、cpu等内容。其中mem_init（内存初始化）做的工作是初始化一个mem_map数组来存储内存地址是否使用。

#### 小tips
cs:ip寻址方式和保护模式的寻址方式的差别
1. cs:ip模式cs寄存器里面直接存的是指令内存地址，而保护模式下cs寄存器下存放的是gdt table的数组下标。保护模式的cs寄存器被称为selector。
