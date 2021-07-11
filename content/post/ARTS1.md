---
title: "ARTS #1"
date: 2021-07-07T20:25:15+08:00
---


## Algorithm
[35\. 搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/)
暴力解法：
```
class Solution(object):
    def searchInsert(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        for i in range(len(nums)):
            if target == nums[i]:
                return i
            if target < nums[i]:
                return i
        return len(nums)
```
自己写的丑陋二分法：
```
class Solution(object):
    def searchInsert(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        result = self.dichotomy(0, len(nums)-1, nums, target)
        return result

    def dichotomy(self, start, end, nums, target):
        if nums[start] == target:
            return start
        if nums[end] == target:
            return end
        if start == (end-1) or start == end:
            if nums[start] > target:
                return start
            if nums[end] < target:
                return end + 1
            else:
                return end
        middle = int((start + end)/2)
        if nums[middle] == target:
            return middle
        if nums[middle] < target:
            return self.dichotomy(middle, end, nums, target)
        if nums[middle] > target:
            return self.dichotomy(start, middle, nums, target)
```
官方的二分法：
```
class Solution(object):
    def searchInsert(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        length = len(nums)
        left = 0
        right = length -1 
        ans = length
        while(left <= right):
            middle = int((left + right)/2)
            if target <= nums[middle]:
                ans = middle
                right = middle - 1
            else:
                left = middle + 1 
        return ans
```
关键点：middle要+1
## Review
[Go Concurrency Patterns](https://talks.golang.org/2012/concurrency.slide#1)
go实现并发的最最核心的观点
1. Don't communicate by sharing memory, share memory by communicating
2. Goroutines and channels are big ideas. They're tools for program construction.They're fun to play with, but don't overuse these ideas.

文章概述：
- Concurrency is not parallelism 并发不是并行 这个程序员应该很清楚了 并发只是cpu的不断切换，本质还是只有一个内存并行 只有多核情况下才能实现并行
- go期望能让程序员简单易懂的实现程序并发
- 本质是实现CSP模型
- goroutine有自己的stack 比thread轻便 
- 基础的channel没有buffer 会互相堵塞
- 几个go并发的例子
## Tip
golang的协程与管道正式在项目中使用
```
//主线程中业务代码
func BatchRhinoTaskDebug(rhinoDebugInfos *[]rhino.RhinoDebugInfo) []rhino.DebugLogResult {
	debugResultChan := make(chan rhino.DebugLogResult)
    // 开启多个协程进行业务处理
	for _, rhinoDebugInfo := range *rhinoDebugInfos {
		rhinoDebugInfo := rhinoDebugInfo
		go func() {
			RhinoTaskDebug(rhinoDebugInfo, debugResultChan)
		}()
	}
    // 主线程中读取channel内容
	var debugResults []rhino.DebugLogResult
	for i := 0; i < len(*rhinoDebugInfos); i++ {
        // 因为声明的管道是堵塞式，只有子协程中往管道放入内容，主线程才能继续for循环
		resultTemp := <-debugResultChan
		debugResults = append(debugResults, resultTemp)
	}
	return debugResults
}
// 协程的具体业务逻辑
func RhinoTaskDebug(rhinoDebugInfo rhino.RhinoDebugInfo, resultChannel chan rhino.DebugLogResult) {
	var err error
	debugLogResult := rhino.DebugLogResult{}
    // 在defer中统一将业务处理结果写入管道中 这样子主线程的管道堵塞处就能继续处理
	defer func() {
		debugLogResult.ResultError = err
		resultChannel <- debugLogResult
	}()
	// 具体的业务代码，生成结果
	debugResult, err := extractDebugResult(debugLog)
	if err != nil {
		return
	}
	debugLogResult = *debugResult
}


```
## Share
分享一篇文章，关于golang是“值传递”还是“引用传递” 同时加上自己的理解
[Go 到底是传值还是传引用](https://mp.weixin.qq.com/s/dLeLqzoz00XypuqzYcwLiA)
### 结论
- go永远是值传递
  - 如果传的是string或者int 则传递给函数的是string或者int的副本（查看内存指针可以发现是不一样的） 在调用函数内部改变副本，不会对原值产生任何影响
  - 如果传的是指针 则传递给函数的是指针地址的复制值副本 在函数内部改变该指针的值，也不会对原值（指针）产生任何影响 但是可能可以指针直接改写内存块内存的值
  - map 和 slice 传递的时候，在函数中对map和slice的修改会影响到原值，但是这个行为类似于指针：它们是包含指向底层 map 或 slice 数据的指针的描述符。go的runtime对map进行了相关封装，在传递map的时候相等于传递了map对应的指针
### 代码样例
#### 指针传递
```
func HelloFunc() {
	// s := "测试指针传递"
	// fmt.Println("传递前内存地址：", &s)
	// fmt.Println("传递前string的内容：", s)
	// hello(&s)
	// fmt.Println("传递后内存地址：", &s)
	// fmt.Println("传递后string的内容：", s)

	s := "测试值传递"
	fmt.Println("传递前内存地址：", &s)
	fmt.Println("传递前string的内容：", s)
	hello1(s)
	fmt.Println("传递后内存地址：", &s)
	fmt.Println("传递后string的内容：", s)
}

func hello1(s string) {
	fmt.Println("传递进来的string内容：", s)
	fmt.Println("传递进来的string的内存地址：", &s)

	s = "直接修改传递的副本的值"

	fmt.Println("修改后的string的内容：", s)
	fmt.Println("修改后的string的内存地址：", &s)
}

// 传递前内存地址： 0xc000012cf0
// 传递前string的内容： 测试值传递
// 传递进来的string内容： 测试值传递
// 传递进来的string的内存地址： 0xc000012d10
// 修改后的string的内容： 直接修改传递的副本的值
// 修改后的string的内存地址： 0xc000012d10
// 传递后内存地址： 0xc000012cf0
// 传递后string的内容： 测试值传递
```
#### 值传递
```
func HelloFunc() {
	s := "测试指针传递"
	fmt.Println("传递前内存地址：", &s)
	fmt.Println("传递前string的内容：", s)
	hello(&s)
	fmt.Println("传递后内存地址：", &s)
	fmt.Println("传递后string的内容：", s)

	// s := "测试值传递"
	// fmt.Println("传递前内存地址：", &s)
	// fmt.Println("传递前string的内容：", s)
	// hello1(s)
	// fmt.Println("传递后内存地址：", &s)
	// fmt.Println("传递后string的内容：", s)
}

func hello(s *string) {
	fmt.Println("传递进来的指针：", s)
	fmt.Println("传递进来的string内容：", *s)
	fmt.Println("传递进来的指针的内存地址：", &s)

	*s = "通过指针修改后的string内容"

	fmt.Println("修改后的指针：", s)
	fmt.Println("修改后的string内容：", *s)
	fmt.Println("修改后的指针的内存地址：", &s)
}
// 传递前内存地址： 0xc000098cd0
// 传递前string的内容： 测试指针传递
// 传递进来的指针： 0xc000098cd0
// 传递进来的string内容： 测试指针传递
// 传递进来的指针的内存地址： 0xc0000b2020
// 修改后的指针： 0xc000098cd0
// 修改后的string内容： 通过指针修改后的string内容
// 修改后的指针的内存地址： 0xc0000b2020
// 传递后内存地址： 0xc000098cd0
// 传递后string的内容： 通过指针修改后的string内容
```
#### map传递
```
func MapFunc() {
	m := make(map[string]string)
	m["测试key"] = "修改前value"
	fmt.Printf("传递前的内存地址：%p\n", &m)
	fmt.Printf("传递前的map内容：%v\n", m)
	mapHello(m)

	fmt.Printf("传递后的内存地址：%p\n", &m)
	fmt.Printf("传递后的map内容：%v\n", m)
}

func mapHello(p map[string]string) {
	fmt.Printf("传递进来的内存地址：%p\n", &p)
	fmt.Printf("传递进来的map内容：%v\n", p)
	p["测试key"] = "修改后value"
	fmt.Printf("修改后的内存地址：%p\n", &p)
	fmt.Printf("修改后的map内容：%v\n", p)
}
// 传递前的内存地址：0xc000010028
// 传递前的map内容：map[测试key:修改前value]
// 传递进来的内存地址：0xc000010038
// 传递进来的map内容：map[测试key:修改前value]
// 修改后的内存地址：0xc000010038
// 修改后的map内容：map[测试key:修改后value]
// 传递后的内存地址：0xc000010028
// 传递后的map内容：map[测试key:修改后value]
```


本周工作心得：
- 调SDK接口一定要认真看下源码 不然很容易出现莫名其妙的问题
- 一个问题花了半天时间定位不出来 先暂停半天 再回过头来，会有意想不到的思路