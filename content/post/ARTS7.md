---
title: "ARTS7"
date: 2021-07-18T15:48:06+08:00
---

## Algorithm
#### [剑指 Offer 06\. 从尾到头打印链表](https://leetcode-cn.com/problems/cong-wei-dao-tou-da-yin-lian-biao-lcof/)
简单的反转单向链表，之前面试字节的时候遇到过，结果当时没做出来。。
这里比官方解法少一点时间复杂度。
```
package reverse_print

// Definition for singly-linked list.
type ListNode struct {
	Val  int
	Next *ListNode
}

func reversePrint(head *ListNode) []int {
	if head == nil {
		return nil
	}
	var next *ListNode
	var prev *ListNode
	for true {
		if head.Next != nil {
			next = head.Next
			head.Next = prev
			prev = head
			head = next
			continue
		}
		head.Next = prev
		break
	}
	var result []int
	for true {
		result = append(result, head.Val)
		if head.Next != nil {
			head = head.Next
			continue
		}
		break
	}
	return result
}
```
## Review
[How To Avoid Retry Storms In Distributed Systems](https://iamkanikamodi.medium.com/how-to-avoid-retry-storms-in-distributed-systems-91bf34f43c7f)
文章主要讲了以下几点：
1. 什么是重试风暴：级联层次较深的分布式系统里面，如果大家都默认请求失败重试3次的话，最终到底层服务的重试次数就会变成指数级别的请求量。如果最底层的服务挂了，这时候整个系统就会充斥着大量无用的重试请求。
2. 如何避免重试风暴：重点就是要从全局角度考虑重试次数，而不是每个请求只考虑自己的重试次数。文章建议是可以使用令牌桶算法进行全局请求限流，正常请求和重试请求都一样要先申请token才能进行请求。
3. 最后文章给了几个避免重试风暴的几个建议：
  - 限制重试速率，这里就是第二点说的全局限速
  - 当依赖方已经明确过载了，就不要再重试了。 
  - 重试次数建议设置成1次
  - 重试不要简单的retry，可以使用退避算法（和退避窗口一样，每次重试之间的时间间隔越来越久）和抖动算法
  - 自适应重试策略，下游健康的时候请求可以放大 下游失败的话，就限制对下游的请求 我理解就是对令牌桶的token大小进行调整
  - 被依赖方最好明确的说明哪些失败需要重试，哪些不建议重试

这篇文章看了还是比较有体会的，前一段时间做需求就遇到过下游依赖方性能比较差，而我一开始就是默认失败了就重试3次，后面发现引发了下游的雪崩。最后自己用Golang的channel实现了一个全局限流队列。

## Tip
本周在单元测试的实践中捞到了一个好用的Golang httpmock库，并写了一篇学习总结：[Golang单元测试实战：httpMock](https://www.jianshu.com/p/545963b593de)

## Share
分享一下阮一峰老师的 [汇编语言入门教程](http://www.ruanyifeng.com/blog/2018/01/assembly-language-primer.html)
由于大学的“勤学刻苦”导致科班出身的本人基础却很差，所以看这篇文章还是学到了蛮多基础知识，解答了一些之前看技术文章困惑的点。
1. 寄存器是cpu内部的存储单元，用来储存最常用的数据，那些最频繁读写的数据（比如循环变量），都会放在寄存器里面，CPU 优先读写寄存器，再由寄存器跟内存交换数据。
2. x86寄存器名称有8个，前面7个寄存器都是通用的，最后一个ESP寄存器有特定用途：保存当前Stack的地址。
3. 用户主动请求而划分出来的内存区域，叫做 Heap（堆）。它由起始地址开始，从低位（地址）向高位（地址）增长。Heap 的一个重要特点就是不会自动消失，必须手动释放，或者由垃圾回收机制来回收
4. Stack 是由于函数运行而临时占用的内存区域。
  - 每次执行call函数调用，都会新建一个帧，用来存储内部变量（这就实现了函数的本地变量）
  - 每一次函数执行结束，就自动释放一个帧，所有函数执行结束，整个 Stack 就都释放了。
  - 等到函数调用结束，对应帧就会被回收，同时系统会回到调用方函数刚刚中断执行的地方，继续往下执行。
  - 栈先进后出，从内存的高位向地位增长。
5. 具体指令：
  - 根据约定，程序从_main标签开始执行，这时会在 Stack 上为main建立一个帧，并将 Stack 所指向的地址，写入 ESP 寄存器。后面如果有数据要写入main这个帧，就会写在 ESP 寄存器所保存的地址
  - push指令用于将运算子放入 Stack。
  - push指令有一个前置操作。它会先取出 ESP 寄存器里面的地址，将其减去N个字节（由算子类型决定），然后将新地址写入 ESP 寄存器。
  - call   _add_a_and_b表示调用add_a_and_b函数。这时，程序就会去找_add_a_and_b标签，并为该函数建立一个新的帧
  - push   %ebx表示将 EBX 寄存器里面的值，写入_add_a_and_b这个帧。这是因为后面要用到这个寄存器，就先把里面的值取出来，用完后再写回去
  - mov指令用于将一个值写入某个寄存器。mov    %eax, [%esp+8] 这一行代码表示，先将 ESP 寄存器里面的地址加上8个字节，得到一个新的地址，然后按照这个地址在 Stack 取出数据。再将取出的数据写入 EAX 寄存器
  - add指令用于将两个运算子相加，并将结果写入第一个运算子。
  - pop指令用于取出 Stack 最近一个写入的值（即最低位地址的值），并将这个值写入运算子指定的位置。pop指令还会将 ESP 寄存器里面的地址加4，即回收4个字节

```
int add_a_and_b(int a, int b) {
   return a + b;
}

int main() {
   return add_a_and_b(2, 3);
}

_add_a_and_b:
   push   %ebx
   mov    %eax, [%esp+8] 
   mov    %ebx, [%esp+12]
   add    %eax, %ebx 
   pop    %ebx 
   ret  

_main:
   push   3
   push   2
   call   _add_a_and_b 
   add    %esp, 8
   ret
```