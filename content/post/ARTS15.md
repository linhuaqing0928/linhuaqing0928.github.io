---
title: "ARTS15"
date: 2021-09-21T08:33:09+08:00
---

## Algorithm
#### [剑指 Offer 12\. 矩阵中的路径](https://leetcode-cn.com/problems/ju-zhen-zhong-de-lu-jing-lcof/)
自己用stack的方式解法，思路和官方是一样的，但是一直不能ac，后面留时间再debug下。
贴一下Golang实现的官方解题：
```
func exist(board [][]byte, word string) bool {
	row := 0
	for row < len(board) {
		column := 0
		for column < len(board[0]) {
			if dfs(row, column, 0, &board, word) {
				return true
			}
			column++
		}
		row++
	}
	return false
}

func dfs(i int, j int, k int, board *[][]byte, word string) bool {
	if !(0 <= i && i < len(*board)) || !(0 <= j && j < len((*board)[0])) || ((*board)[i][j] != word[k]) {
		return false
	}
	if k == len(word)-1 {
		return true
	}
	(*board)[i][j] = ' '
	res := (dfs(i+1, j, k+1, board, word) || dfs(i-1, j, k+1, board, word) || dfs(i, j+1, k+1, board, word) || dfs(i, j-1, k+1, board, word))
	(*board)[i][j] = word[k]
	return res
}
```
## Review
[A New, Simpler Way to Do Dependency Injection in Go](https://elliotchance.medium.com/a-new-simpler-way-to-do-dependency-injection-in-go-9e191bef50d5)
文章用一个例子介绍了DI的好处、provider模式的应用，最后介绍了一个开源框架dingo。dingo有点像spring，使用yaml进行依赖注入配置，然后生成provider。和官方[wire](https://github.com/google/wire)思路一致，推荐使用wire。
## Tip
这周做双11大促压测保障的时候，第一次接触到了业务降级预案演练的概念。之前大促压测流程保障基本都是比较简单的：
1. 定制一个压测目标。
1. 然后进行压测。
1. 达到目标之后，在入口处设置限流，超过限流值的请求都不再能够访问服务。从而防止超过性能上限的流量打入服务导致雪崩。

这种解决方案其实是比较简单粗暴的，产品业务同学需要做的只是：
1. 确定业务最佳体验方案然后研发相对应的进行代码开发。
2. 性能达到瓶颈之后，一般都是进行服务器资源扩容或者研发同学进行代码性能优化。

而服务降级则是另一个思路：
产品业务同学在最佳体验方案之外，需要准备几个有损降级方案。这里的有损降级就是在请求qps达到某个阈值之后
1. 代码内部针对性的跳过一些非核心处理逻辑，只提供一些核心功能。例如原来可能要返回订单详情和相对应的推荐产品，降级之后只返回订单详情。
2. 或者减少返回内容丰富度。例如一个推荐系统，降级之前可能需要推荐10个信息，降级之后推荐4个信息。
3. 又或者直接返回某个default值，然后对应的业务处理放入异步队列中从而达到削峰的目的。
4. 等等等

简单总结一下就是：降级需要从全局角度针对业务特性进行设计，然后通过提供可接受的有损服务来提高服务整体可承担的请求qps数。
## Share
[深度剖析：Redis分布式锁到底安全吗？看完这篇文章彻底懂了！](https://mp.weixin.qq.com/s/s8xjm1ZCKIoTGT3DCVA4aw)
很好的一篇文章，深入的分析了分布式锁的存在原因、普通的redis分布式锁的优缺点、针对优缺点又衍生出什么解决方案。
1. 死锁：设置过期时间
1. 过期时间评估不好，锁提前过期：守护线程，自动续期
1. 锁被别人释放：锁写入唯一标识，释放锁先检查标识，再释放
1. redis集群模式下，主从切换场景的安全问题。介绍了redlock方案以及对应的争论。