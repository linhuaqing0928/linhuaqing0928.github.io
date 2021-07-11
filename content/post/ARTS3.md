---
title: "ARTS #3"
date: 2021-07-07T20:34:44+08:00
---

## Algorithm
#### [剑指 Offer 04\. 二维数组中的查找](https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof/)
这道题是看了官方解法才解出来的，我个人的解题思路考虑了以下几个场景：
1. 暴力解法，发现会超时
2. 暴力接法的基础上想到了二分法提供性能，但是想到这个题目的侧重点应该不是二分法，所以就没有正式动手。
3. 联想到数组是有序的，所以开始考虑从左上角出发进行寻找，但是左上角需要考虑回退和绕圈的场景，比较复杂一直没有做出来。
暴力递归解法：
```
class Solution(object):
    def findNumberIn2DArray(self, matrix, target):
        """
        :type matrix: List[List[int]]
        :type target: int
        :rtype: bool
        """
        for row in matrix:
            for column in row:
                if column==target:
                    return True
        return False
```
线性查找：
```
class Solution(object):
    def findNumberIn2DArray(self, matrix, target):
        """
        :type matrix: List[List[int]]
        :type target: int
        :rtype: bool
        """
        row_nums = len(matrix)
        if row_nums == 0:
            return False
        column_nums = len(matrix[0])
        if column_nums ==0:
            return False
        current_column = column_nums-1
        current_row = 0
        while True: 
            if current_column < 0  or current_row >= row_nums:
                return False
            current_value = matrix[current_row][current_column]
            if current_value == target:
                return True
            if current_value < target:
                current_row += 1
                continue
                    
            if current_value > target:
                current_column -= 1
                continue
```
## Review
[FIT: Failure Injection Testing](https://netflixtechblog.com/fit-failure-injection-testing-35d8e2a9bb2)
文章介绍了netflix的FIT（故障注入测试），大概内容如下：
1. FIT的由来：混沌工程一般都是整个服务注入延迟、网络不可用等，对线上用户影响太大了 FIT可以理解为是一个小型、可控制的monkey，影响面不像混沌测试那么大。同时因为FIT是可控制的，所以可以慢慢的放大测试范围，如果最后FIT测试通过就可以直接上monkey
2. FIT的原理：
  - FIT会给各个中间件推送故障模拟metadata 这些metadata我理解应该是包括：哪些request在匹配范围、针对这些匹配范围内的请求应该注入什么样的故障
  - 网关zuul接收到的所有request都和metadata一一进行匹配，如果匹配到了就在request context里面接入对应的故障内容
  - 后续的各个中间件（熔断限流、缓存中间件、DB持久层框架等等）会识别request context中的故障内容，对应的进行一些特殊故障注入操作。比如：直接丢弃报文、增加延时等等
3. 还需要一个trace系统来跟踪这些request 然后把所有这些注入点（中间件）组合成一个测试场景。 我理解这样子既是输入也是输出。

整篇文章看来下，感觉要实现可控的故障注入 需要整个公司的基础组件都一起配合才能实现，是一个很大的工程。 后续深入了解一下字节的混沌平台，以及有没有和FIT类似的平台。
## Tip
golang使用time.After和channel实现超时机制。
default中是具体的业务代码，我这里是websocket的byte读取，可以替换成对应的业务。
需要注意的是，这里的done channel需要是缓存队列，不然select就会堵塞住。
```
done := make(chan bool, 1)
timeOut := time.After(time.Second * 10)

	for {
		select {
		case <-done:
			logs.Info("debug done!")
			return buffer.String(), nil
		case <-timeOut:
			logs.Error("get debug log time out!debugID:%s", debugID)
			return "", errors.New("get debug log time out!")
		default:
			_, message, err1 := c.ReadMessage()
			if err1 != nil {
				log.Println("read:", err1)
				logs.Info("read complete! debugID: %s, rhinoDebugInfo:%v", debugID, rhinoDebugInfo)
				done <- true
			}
			buffer.Write(message)
		}
	}
```
## Share
[上帝视角看进程调度](https://mp.weixin.qq.com/s/zzGcNr59AJ3bqI9GF9xMqA)
这片文章详细的讲了进程切换的过程，之前只是知道是进程各自申请各自的时间片，到点之后CPU会重新进行调度，但是具体如何调度没有搞的特别清楚，看了这篇文章就比较清楚了。
copy一下文中的总结：
1. 罪魁祸首的，就是那个每 10ms 触发一次的定时器滴答。
2. 而这个滴答将会给 CPU 产生一个时钟中断信号。
3. 而这个中断信号会使 CPU 查找中断向量表，找到操作系统写好的一个时钟中断处理函数 do_timer。
4. do_timer 会首先将当前进程的 counter 变量 -1，如果 counter 此时仍然大于 0，则就此结束。
5. 但如果 counter = 0 了，就开始进行进程的调度。
6. 进程调度就是找到所有处于 RUNNABLE 状态的进程，并找到一个 counter 值最大的进程，把它丢进 switch_to 函数的入参里。
7. switch_to 这个终极函数，会保存当前进程上下文，恢复要跳转到的这个进程的上下文，同时使得 CPU 跳转到这个进程的偏移地址处。
8. 接着，这个进程就舒舒服服地运行了起来，等待着下一次滴答的来临。