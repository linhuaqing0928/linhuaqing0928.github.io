---
title: "ARTS12"
date: 2021-08-22T22:35:07+08:00
---

## Algorithm
#### [剑指 Offer 11\. 旋转数组的最小数字](https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/)
1. 解法1：暴力破解法，从前往后遍历，如果index+1比index小就return
2. 解法2：二分法。关键点就是数组本身是一个旋转数组，也就是半递增数组。所以以中间一个最小值点为边界，最小值左右侧都是递增的。通过拿mid点和最右侧点进行对比：
  - 如果mid比right小，说明最小值不在右侧。
  - 如果mid比right大，说明最小值不在左侧。
  - 如果mid和right一样大小，说明存在重复元素。由于我们不知道重复元素有多少个，所以我们right--即可。
```
// 暴力遍历，时间复杂度O(n)
func minArray1(numbers []int) int {
	length := len(numbers)
	index := 0
	for index < length-1 {
		if numbers[index] > numbers[index+1] {
			return numbers[index+1]
		}
		index++
	}
	return numbers[0]
}

// 二分法，时间复杂度O(logn)
// 关键在于
func minArray(numbers []int) int {
	left := 0
	right := len(numbers) - 1
	for left < right {
		mid := left + (right-left)/2
		if numbers[mid] < numbers[right] {
			right = mid
			continue
		}
		if numbers[mid] > numbers[right] {
			left = mid + 1
			continue
		}
		if numbers[mid] == numbers[right] {
			right--
			continue
		}
	}
	return numbers[left]
}
```
## Review
[Go Pointers: Why I Use Interfaces (in Go)](https://krancour.medium.com/go-pointers-why-i-use-interfaces-in-go-338ae0bdc9e4) 文章举了一个在Golang中使用interface的例子，来安利我们“面向interface编程“。
1. Golang中没有构造函数的说法，只有struct。
2. 为了实现类似其他语言中的构造函数，我们一般是新增一个类似的构造函数
 ```
type Widget struct {
	id string
}

func NewWidget() Widget {
	return Widget{
		id: uuid.NewV4().String(),
	}
}

func (w Widget) ID() string {
	return w.id
}
```
3. 但是这时候因为struct本身是public（大写的），调用方还是可以直接new Widget{}的方式来绕过NewWidget方法。为了防止这个情况，我们通常会将struct设置为private（改为小写），来防止绕过
```
type widget struct {
	id string
}

func NewWidget() widget {
	return widget{
		id: uuid.NewV4().String(),
	}
}

func (w widget) ID() string {
	return w.id
}
```
4. 这时候就出现一个问题了，因为widget是小写的，虽然编译器不会判定报错，但是godoc在生成文档的时候就不会生成widget相关的内容和注释。就导致你对外暴露了一个实例，但是文档中找不到任务关于这个实例对应结构体的信息和注释。这是很不推荐的。
5. 如果我们用interface来改造一下，就能避免这个问题。
```
// Widget is a ...
type Widget interface {
	// ID returns a widget's unique identifier
	ID() string
}

type widget struct {
	id string
}

// NewWidget() returns a new Widget
func NewWidget() Widget {
	return widget{
		id: uuid.NewV4().String(),
	}
}

func (w widget) ID() string {
	return w.id
}
```
评论区有人贴出了Golang官方wiki的code review建议：[Interfaces](https://github.com/golang/go/wiki/CodeReviewComments#interfaces)，我个人是比较赞同code review上的建议：**不要在生产者（也就是具体struct类所在的包）里面定义interface，而是应该要在消费者所在的包定义interface**。 
不过这2个文章本身说的不是一个事情，前者是通过interface在Golang中实现构造函数，后者则是对interface定义位置的建议。各取所需就可以。
## Tip
推荐一个Golang的配置文件读取开源库：[viper](github.com/spf13/viper)，支持json、yaml、配置中心、buffer字节流等等配置读取，很好用。
## Share
[Go 依赖注入：为什么把 dig 迁移到 wire](https://mp.weixin.qq.com/s/bHXRSpiIhycoQLN5oz0QMA)

[一文读懂 Go官方的 Wire](https://mp.weixin.qq.com/s/ZQKi9O7DRJ3qGWhDL9aTVg)

分享Golang官方依赖注入的工具wire，有如下好处：
1. 通过代码生成的方式实现依赖注入，如果有依赖缺失bug在编译期间就能发现问题。而如果使用的是dig框架这种运行时框架，就只能在运行时暴露问题。
2. 依赖关系在代码上生成后存在源码，方便静态分析等工具分析。
3. 节省人工手写依赖注入代码的时间。

个人想法：
1. Clear is better than clever ，Reflection is never clear。 大道至简，简单优雅的代码比花式炫技更需要功力。可以看到Golang官方的wire、go mock等都是基于这一点进行设计。
2. 依赖注入很重要，工作过程要时刻谨记。
3. 善用工具。