---
title: "ARTS #4"
date: 2021-07-07T20:34:48+08:00
---

## Algorithm
#### [剑指 Offer 10- I. 斐波那契数列](https://leetcode-cn.com/problems/fei-bo-na-qi-shu-lie-lcof/)
key：动态规划、python不用考虑int范围
```
class Solution(object):
    def fib(self, n):
        """
        :type n: int
        :rtype: int
        """
        if n == 0:
            return 0
        if n == 1:
            return 1
        pre1 = 0
        pre2 = 1
        index = 2
        while index <= n:
            pre1 , pre2 = pre2, pre1 + pre2
            index = index + 1
        return pre2 % 1000000007
```
## Review
[SOLID Go Design](https://dave.cheney.net/2016/08/20/solid-go-design)
这篇文章讲了solid原则在go上一些最佳实践的点，主要内容如下：
- 单一原则： 在go程序要做到：每个 Go package 本身就是一个小的 Go 程序，一个单一的变更单元，具有单一的责任。从package的命名、实现的内容都要做到单一功能。
  - 这里特意提到了unix的设计理念：small, sharp tools which combine to solve larger tasks, oftentimes tasks which were not envisioned by the original authors
- 开闭原则：也就是Software entities should be open for extension, but closed for modification。go本身不支持函数重载。很好的做到了being open for extension, are closed for modification
- 里式替换原则：go本身没有抽象类的概念，但是interface很好的支持了替换原则。
- 接口隔离原则：Clients should not be forced to depend on methods they do not use。  在go里面的最佳实践：A great rule of thumb for Go is accept interfaces, return structs.
- 依赖注入原则（Dependency Inversion Principle）：High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.   这里提到在go里面做到依赖注入原则的一个衡量标准就是：If you have a package whose functions cannot operate without enlisting the aid of another package, that is perhaps a sign that code is not well factored along package boundaries.

个人感觉在目前一般的业务代码开发中（特别是测开项目）对性能的要求其实不是那么高，反而对代码的可读性要求比较高。有一句话叫做：代码是给人看的，不是给机器看的。 而solid原则就是让我们能写出可读性高，容易重构的代码。review了下自己最近写的go代码，确实一团糟，读了这篇文章，以后尽量做到：
1. package尽量小而单一职责。不要感觉看着功能差不多的代码就都堆到一个package里面。
2. 多用组合，少用继承。多用interface，少用struct。
3. A great rule of thumb for Go is accept interfaces, return structs
4. package之间尽量独立解耦，不要互相依赖。有依赖的话，也尽量做到通过interface依赖注入，而不是package内部直接实例化使用。
5. 现在代码开发为了图方便，struct内部属性全都大写对外开放，但是这样子其实是不对的 要尽量做到开闭原则。

## Tip
Golang如何在application运行过程查看goroutine、heap使用情况.
1. 引入_ "net/http/pprof"
2. 在main函数最开始的地方，开启一个goroutine进行pprof监听
3. 运行main函数之后，通过浏览器打开http://127.0.0.1:9876/debug/pprof/ 其中端口号替换为代码中填写的端口号即可看到application的内存等使用情况。
```
import (
	_ "net/http/pprof"

	"fmt"
	"net/http"
)

func main() {
	// goroutine数量监控，正式上线需要注释掉
	go func() {
		fmt.Println("pprof start...")
		fmt.Println(http.ListenAndServe(":9876", nil))
	}()

	gin.Init()

	r := gin.Default()

	// waitFunc, mw := shutdown.New(10e9, 0)
	// r.Use(mw)
	// defer waitFunc()

	register(r)

	r.Run()

}
```

## Share
[Go: GC 是怎样监听你的应用的？](https://mp.weixin.qq.com/s/Nqr096q51NWbyN6VujFTzA)
文章主要内容：
1. GC会在堆内存占用douoble的时候被触发。
2. 如果条件1在2分钟内没有触发，那么GC会在距离上一次GC结束时间的2分钟的时候触发一次
3. 为了方便跟踪标记内存的使用情况，在GC触发的时候 GC会为每一个CPU分配一个处于 sleep 状态的 goroutine。

GC还是比较难，这篇文章除了以上3点，其他的看的有点云里雾里。比如怎么标记内存使用（三色标记？）、标记阶段到底在做啥等等。 这里记一个Todo，后续再深入了解一下go得GC。