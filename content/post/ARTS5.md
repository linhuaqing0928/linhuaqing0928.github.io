---
title: "ARTS #5"
date: 2021-07-07T20:34:51+08:00
---








## Algorithm
 [剑指 Offer 09\. 用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

看了官方解答之后才做出来的。栈A存push的内容，栈B存remove的内容。栈B空的时候，把栈A的内容一一remove出来，然后再一一push到栈B里面。

```
type CQueue struct {
	stack1, stack2 *list.List
}

func Constructor() CQueue {
	return CQueue{
		list.New(),
		list.New(),
	}
}

func (this *CQueue) AppendTail(value int) {
	this.stack1.PushBack(value)
}

func (this *CQueue) DeleteHead() int {
	if this.stack2.Len() > 0 {
		back := this.stack2.Back()
		element := this.stack2.Remove(back)
		return element.(int)
	}
	index := 0
	for index < this.stack1.Len() {
		back := this.stack1.Back()
		element := this.stack1.Remove(back)
		this.stack2.PushBack(element)
	}
	if this.stack2.Len() == 0 {
		return -1
	}
	back := this.stack2.Back()
	element := this.stack2.Remove(back)
	return element.(int)
}
```
## Review
[Using Numba for blazing performance in Python](https://umangshrestha09.medium.com/using-numba-for-blazing-performance-in-python-656e8e32f8c)
文章展示了几个用numba进行python性能提升对比的例子

简单来说就是python本身因为是解释性语言，大家平时使用的cpython解释器没有jit（Just-in-time compiler）优化，所以使用起来会感觉python性能比较差。而numba可以针对性的对某段代码进行jit优化，所以性能提升就比较大。

之前看过一篇文章，换一个pypy编译器python性能也是同样飞起。[为什么Python比C++慢很多？](https://www.zhihu.com/question/62185153)
```
import numpy as np
import time
import math
from numba import jit

size = int(5e7)
array = np.arange(size).astype(np.float32)

def function_with_normal_loop(array):
    out = []
    for i in array:
        out.append(math.sqrt(i))
    return out

def function_with_list_comprehension(array):
    return [math.sqrt(x) for x in array]

def function_with_map(array):
    return list(map(math.sqrt, array))

@jit(nopython=True)
def function_with_normal_loop_jit(array):
    out = []
    for i in array:
        out.append(math.sqrt(i))
    return out

@jit(nopython=True)
def function_with_list_comprehension_jit(array):
    return [math.sqrt(x) for x in array]

@jit(nopython=True)
def function_with_map_jit(array):
    return list(map(math.sqrt, array))

def culculate_time(f, array):
    start = time.time()
    f(array)
    end = time.time()
    print(end-start)
    
culculate_time(function_with_normal_loop,array)
culculate_time(function_with_list_comprehension,array)
culculate_time(function_with_map,array)
culculate_time(function_with_normal_loop_jit,array)
culculate_time(function_with_list_comprehension_jit,array)
culculate_time(function_with_map_jit,array)

# 10.340049028396606
# 8.859677076339722
# 5.530158042907715
# 2.5269722938537598
# 2.5843582153320312
# 3.2769558429718018
```

## Tip
[https://github.com/tidwall/gjson](https://github.com/tidwall/gjson)
这周工作过程用了一个很好用的golang json解析开源库 gjson，分享一下，使用方法在github readme中都有写。

## Share

分享一篇讲解Golang timer的文章：[Go timer 是如何被调度的？](https://mp.weixin.qq.com/s/eRWPdWWBDyc9TZHB3u197Q)

总结下大概内容：
1. Go 1.14 版本之后，每个 P 单独维护一个四叉堆。减少了 Goroutine 之间的并发问题，提升了 timer 了性能
2. timer的调度一般有以下几种方式：
  - 业务方调用NewTimer，timer.After, timer.AfterFunc 生产 timer, 加入对应的 P 的堆上
  - 业务方调用 timer.Stop, timer.Reset 改变对应的 timer 的状态。然后等待GMP调度周期进行其他移除或者重新加入四叉堆操作。
  - GMP 在调度周期内中会调用 checkTimers ，遍历该 P 的 timer 堆上的元素，根据对应 timer 的状态执行真的操作。-- 在timer.reset函数中，等待被触发的timer就会通过改变timer的状态，然后让GMP调度来重新将timer加入四叉堆上
3. timer被Stop时只是修改了timer的状态位timerDeleted 然后等待GMP触发adjusttimers 或者 runtimer来将timer重四叉堆上移除
4. timer真正执行者是GMP 调度周期内，GMP通过 runtime.checkTimers 调用 timer.runtimer()判断timer能否被触发 如果能被触发，就会通过回调函数 sendTime 给 Timer 的 channel C 发一个当前时间，告诉我们这个 timer 已经被触发。 如果是ticker的话，被触发后，会重新加入四叉堆。

ps：
timer的源码解读：[timer源码解读](https://mp.weixin.qq.com/s/a481CXU6VymHTH8OOVimGQ)
GMP的介绍：[GMP介绍](https://segmentfault.com/a/1190000023869478)
下周再继续深入学习一下