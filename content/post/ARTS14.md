---
title: "ARTS14"
date: 2021-09-05T17:45:58+08:00
---

## Algorithm
#### [剑指 Offer 21\. 调整数组顺序使奇数位于偶数前面](https://leetcode-cn.com/problems/diao-zheng-shu-zu-shun-xu-shi-qi-shu-wei-yu-ou-shu-qian-mian-lcof/)
2个解法：
1. 头尾双指针
  - 2个指针分别从头尾出发
  - 判断头指针所在位置的值是奇数还是偶数
  - 如果是奇数，则头指针++然后continue
  - 如果是偶数，则交换头尾指针所在位置数组的数值。然后尾指针--，continue。
2. 快慢双指针
  - 2个指针一起从头部出发
  - 判断快指针所在位置的值是奇数还是偶数
  - 如果是奇数，则交换快慢指针所在位置数组的数值，然后快慢指针都++，continue。
  - 如果是偶数，则快指针++ 然后continue
```
// 首位双指针
func exchange1(nums []int) []int {
	start := 0
	end := len(nums) - 1
	for start < end {
		if nums[start]%2 == 1 {
			start++
			continue
		}
		if nums[start]%2 == 0 {
			temp := nums[start]
			nums[start] = nums[end]
			nums[end] = temp
			end--
			continue
		}
	}
	return nums
}

// 快慢双指针
func exchange(nums []int) []int {
	slow := 0
	fast := 0
	for fast < len(nums) {
		if nums[fast]%2 == 1 {
			temp := nums[fast]
			nums[fast] = nums[slow]
			nums[slow] = temp
			fast++
			slow++
			continue
		}
		if nums[fast]%2 == 0 {
			fast++
			continue
		}
	}
	return nums
}

```
## Review
[Provider Pattern in Go and Why You Should Use It](https://medium.com/swlh/provider-model-in-go-and-why-you-should-use-it-clean-architecture-1d84cfe1b097)
文章讲述了provider模式的好处：
1. 啥是povider模式：2005年起源于微软的一个编程模式。
2. 一个简单的Golang provider模式例子。
3. 使用provider模式更有利于实现Clean Architecture。
4. 但是不管怎么做解耦，最终在我们的provider模式代码中还是需要调用到第三方SDK。这时候还是会造成耦合，怎么解决呢？答案就会面向接口编程，在consumer中用interface的方式调用第三方SDK。

整篇文章看似在讲provider模式，其实主要观点还是通过依赖注入和面向接口编程的方式，将业务代码逻辑和第三方SDK/接口进行解耦。最近一段时间一直在深入学习怎么在Golang中写出具有可测性的代码，目前来看最佳实践可以归纳如下：
1. 在调用第三方SDK/接口（甚至是同一个项目中的service、dao层）的时候，用依赖注入的方式来引入被调用方。而不是在业务代码逻辑中去自行实例化第三方。
2. 依赖注入的过程，我们注入的应该是第三方的抽象interface/func，而不是一个真实具体的第三方结构体。这样子，调用方和调用方就能够解耦开。同时我们调用第三方的这部分业务逻辑代码也更有可测性了（Golang中的多态在interface上有效，所以mock/stub前提就是面向interface编程）
3. 在面向接口编程的时候，interface放在producer还是consumer上呢？Golang官方的答案（[Interfaces](https://github.com/golang/go/wiki/CodeReviewComments#interfaces)）就是放在consumer上，这得益于Golang interface的隐式继承的特性。
## Tip
https://github.com/idoubi/sql2struct 建表sql转Golang struct浏览器插件
## Share
[Go select 竟然死锁了。。。](https://mp.weixin.qq.com/s/YB_mBJMAmRruNrb4NPy8sw)
> For all the cases in the statement, the channel operands of receive operations and the channel and right-hand-side expressions of send statements are evaluated exactly once, in source order, upon entering the “select” statement. The result is a set of channels to receive from or send to, and the corresponding values to send. Any side effects in that evaluation will occur irrespective of which (if any) communication operation is selected to proceed. Expressions on the left-hand side of a RecvStmt with a short variable declaration or assignment are not yet evaluated.

Golang在select语法中，不管最终执行哪个case，都会按源码的顺序对每一个 case 子句进行求值：这个求值只针对发送或接收操作的额外表达式。所以如果在<-的右边再嵌套一个<-的话，就会造成泄漏。
同样的，在平时使用timer.After的时候如果在case中把timer.After放置在管道操作符的右边的话，就会造成timer的泄漏。因为timer.After本身也是返回一个<-chan Time。

之前项目开发中的做法是：将timer.After赋值给一个全局变量然后在case中使用该局部变量。
```
timeOut := time.After(time.Second * 300)

	for {
		select {
		case <-done:
			// logs.Info("read complete! debugID: %s, rhinoDebugInfo:%v", debugID, rhinoDebugInfo)
			s.debugLog = buffer.String()
			return nil
		case <-timeOut:
			logs.CtxError(s.ginContext, "get debug log time out!url:%s", url)
			return errors.New("get debug log time out!")
		default:
			_, message, err1 := c.ReadMessage()
			if err1 != nil {
				done <- true
			}
			buffer.Write(message)
		}
	}
```