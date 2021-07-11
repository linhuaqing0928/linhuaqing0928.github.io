---
title: "ARTS6"
date: 2021-07-07T20:34:54+08:00
---

## Algorithm
#### [剑指 Offer 03\. 数组中重复的数字](https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/)

最容易想到的解法就是map不重复算法
```
// map不可重复解法
func findRepeatNumber(nums []int) int {
	unrepeat_map := make(map[int]int)
	for _, num := range nums {
		if _, ok := unrepeat_map[num]; ok {
			return num
		} else {
			unrepeat_map[num] = 1
		}
	}
	return -1
}
```
看评论区有一个原地交换算法，很精妙。充分利用了长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内这一条件，实现如下：
```
// 原地交换算法
func findRepeatNumber1(nums []int) int {
	index := 0
	for index < len(nums) {
		if index == nums[index] {
			index++
			continue
		}
		if nums[nums[index]] == nums[index] {
			return nums[index]
		}
		temp := nums[nums[index]]
		nums[nums[index]] = nums[index]
		nums[index] = temp
	}
	return -1
}
```
## Review
[The Wide World of Software Testing](https://medium.com/@nirespire/the-wide-world-of-software-testing-d38835b8c90e)
文章从上到下介绍了测试金字塔的几个阶段，都是一些概念。很赞同里面对单元测试的几个定义：
- Unit tests should be pretty lightweight and run fast since they don’t depend on anything external to the unit of code being tested
- The only way they can fail is by changing the unit of code they are testing
- It is usually necessary to augment unit tests with more comprehensive testing methods to ensure complete confidence in your code working as expected

## Tip
在Golang怎么判断map是否存在某个key：
```
        if _, ok := unrepeat_map[num]; ok {
			return num
		} else {
			unrepeat_map[num] = 1
		}
```

## Share

 [年度最佳【golang】GMP调度详解](https://segmentfault.com/a/1190000023869478) 
这篇文章介绍了Golang的GMP调度，有浅有深。目前只能看懂前面概念介绍和流程介绍部分，从汇编和源码讲解开始就看的有点懵逼了，这里先mark一下。后续学习了相关知识后再回过头来继续学习这篇文章。
- 核心概念
  - G是goroutine的头文字, goroutine可以解释为受管理的轻量线程
  - M是machine的头文字, 在当前版本的golang中等同于系统线程
  - P是process的头文字, 代表M运行G所需要的资源
- 数据结构
  - ![image.png](https://upload-images.jianshu.io/upload_images/26449847-9fcd18c8d44aae9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  - 每个P拥有自己的本地运行队列来存储G 同时有一个全局运行队列来存储放不下的G。
  - 当M从P的本地运行队列获取G时, 如果发现本地队列为空会尝试从其他P盗取一半的G过来
