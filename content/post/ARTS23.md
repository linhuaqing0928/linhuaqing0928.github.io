---
title: "ARTS23"
date: 2021-11-28T20:07:44+08:00
---

## Algorithm
#### [剑指 Offer 13\. 机器人的运动范围](https://leetcode-cn.com/problems/ji-qi-ren-de-yun-dong-fan-wei-lcof/)
1. 题目关键是它每次可以向左、右、上、下移动一格（不能移动到方格外）。理解了这个之后就是DFS就好了。
1. 遍历过程维护一个map，存已经经过的点。
```
func movingCount(m int, n int, k int) int {
	result := 0
	done := make(map[[2]int]int)
	dfs(0, 0, m, n, k, &result, done)
	return result
}

func dfs(rowIndex, columnIndex, m, n, k int, count *int, done map[[2]int]int) {
	if rowIndex >= m || columnIndex >= n {
		return
	}
	if !enterPermission(rowIndex, columnIndex, k) {
		return
	}
	temp := [2]int{}
	temp[0] = columnIndex
	temp[1] = rowIndex
	_, ok := done[temp]
	if ok {
		return
	}
	done[temp] = 1
	*count++
	dfs(rowIndex, columnIndex+1, m, n, k, count, done)
	dfs(rowIndex+1, columnIndex, m, n, k, count, done)
	return
}

func enterPermission(rowIndex int, columnIndex int, k int) bool {
	return (numberSum(rowIndex) + numberSum(columnIndex)) <= k
}

func numberSum(input int) int {
	a := input / 100
	input = input % 100
	b := input / 10
	input = input % 10
	return a + b + input
}

```
## Review
[Context in Go](https://janteshital.medium.com/context-in-go-language-63cef994ed4b)
文章简单介绍了go标准库的context包，都是一些基本用法，刚好最近做需求频繁的用到context，复习一下。建议直接看标准库源码，注释写的很清晰。
## TIP
这周代码开发过程学会了git stash的使用，场景：
1. 我和A同学共用一个开发分支1
1. A同学在17:00往开发分支1提交了代码
1. 我在18:00准备往开发分支1提交代码 发现和远端代码冲突 这时候无法pull 也无法提交自己的代码

我之前的做法从我的本地再拉一个本地分支2出去，然后本地分支做pull和分支1远端保持一致，最后再往把分支2往分支1合并，在合并过程做冲突解决。其实git本身就提供了一个类似的功能。使用步骤如下：
1. git stash save "xxx" 将本地分支stash保存
1. 执行git stash list可以看到刚刚stash出来的一个stash标签
1. 本地分支做pull 和远端最新保持一致
1. git stash apply stash@{$num} 把stash中的内容应用到本地分支
1. 然后解决本地分支的冲突然后提交到远端就可以了

可以看到其实本质和我们之前手动拉分支是一样的，就是隔离保存然后再解决冲突。用stash更方便快捷。
## Share
前几天逛脉脉的时候看到有人提问（是的，你没看错，脉脉上会有人讨论技术问题。。）：如果在一台单核机器上运行多线程程序，这时候还存在线程安全问题吗？对于我这个虽然软工科班出身，但是大学没上过课的人来说，其实我一时之间是答不上来的。

首先说下我之前对多线程安全的认知是这样的：多线程可能分布在不同核上，而数据是存在内存中的。如果2个线程分别在2个核上运行，如下场景（2个线程分别对同一个变量做++操作）就会有线程安全问题：
1. 核1上的线程1根据地址去内存里面取出变量1的值，于此同时核2上的线程2也并行的根据地址去内存里面取出变量1的值。
1. 线程1对变量1的值做++操作。同时线程2也对变量1的值做++操作。这时候就存在线程安全问题。期望的是，经过一次操作之后变量1应该是+2，但是实际上只有+1。

所以之前在我的认知里面，线程安全问题是因为CPU多核能够并行的访问内存，在不加锁的情况下会产生内存安全问题。如果是在单核机器上运行，因为只有一个核就算是多线程也是不断地切换CPU时间片，理论上各线程取到的应该都是最新内存上的数据的 不加锁也不会有内存安全问题。

看了评论区其他人的答复，再加上网上找资料，大概知道了答案。

首先，CPU访问内存不是我想象的直接对内存进行读和写然后计算的操作，因为CPU计算的速度比内存读写快太多了，一般CPU都自带一级缓存和二级缓存以及读写最快的内存寄存器。也就是说，那些最频繁读写的数据（比如循环变量），都会放在寄存器里面，CPU 优先读写寄存器，再由寄存器跟内存交换数据。
其次，单核CPU的多线程切换过程有一个步骤就是要把当前正在运行的线程的上下文（包括正在用的寄存器的值）存到内存里面，等到CPU时间片切换回来的时候再去内存里面把刚刚运行一半的上下文取出来，放回到寄存器里面。
知道了上面2点之后，咱们再回来看咱们开头提的问题。
1. 线程1占用CPU时间片的时候，把变量1的值（假设这时候是10）从内存中取到寄存器中，然后准备在CPU中进行相关计算操作。
1. 这时候发生了线程切换，线程1的上下文（包括刚刚取出来的变量1的值10）会暂时存到另一块内存中。
1. 线程2占用CPU时间片的过程，从内存中读到变量1的值（这时候还是10），然后做了++操作，这时候变量1的值变成11.
1. 轮到线程1的CPU时间片 切换的时候会把刚刚暂存的上下文取回来（这时候变量1的值是旧的10，而不是线程2修改后的11）然后进行++操作，最后写回内存。

这时候就有了多线程并发问题，变量1不是我们期望的12，而是11。