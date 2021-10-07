---
title: "ARTS19"
date: 2021-10-07T19:42:28+08:00
---

## Algorithm
#### [剑指 Offer 14- II. 剪绳子 II](https://leetcode-cn.com/problems/jian-sheng-zi-ii-lcof/)
这道题比剪绳子|难在绳子长度范围可以达到1000，并且需要取模。如果是直接计算的话，就算是int64也会溢出，所以需要循环取模。
```
func cuttingRope(n int) int {
	var result int
	if n == 2 {
		return 1
	}
	if n == 3 {
		return 2
	}
	remind := n % 3
	if remind == 0 {
		// result = int(math.Pow(3, float64(n/3))) % 1000000007
		result = getRemind(n/3) % 1000000007
	}
	if remind == 1 {
		// result = int(math.Pow(3, float64(n/3)-1)) * 4 % 1000000007
		result = getRemind(n/3-1) * 4 % 1000000007
	}
	if remind == 2 {
		// result = int(math.Pow(3, float64(n/3))) * 2 % 1000000007
		result = getRemind(n/3) * 2 % 1000000007
	}
	return result
}

func getRemind(n int) int {
	index := 1
	var result int
	result = 1
	for index <= n {
		result = result * 3 % 1000000007
		index++
	}
	return result
}
```
## Review
[Should We Use Pointers In Go? Truth Is Complicated](https://medium.com/beyn-technology/should-we-use-pointers-in-go-truth-is-complicated-75fccae92f27)
Golang中使用指针虽然能减少内存使用，但是会造成内存逃逸使得变量存放在堆上，并最终影响影响到到GC的耗时。
## Tip
使用go build -gcflags '-m -l' xxx.go可以看到代码的逃逸分析的过程和结果
## Share
[我要在栈上。不，你应该在堆上](https://eddycjy.com/posts/go/talk/2019-05-20-stack-heap/)
Golang里面内存逃逸分析的规则：
1. 是否有在其他地方（非局部）被引用。只要有可能被引用了，那么它一定分配到堆上。否则分配到栈上
2. 即使没有被外部引用，但对象过大，无法存放在栈区上。依然有可能分配到堆上。

还有个特别的点：
1. 当形参为 interface 类型时，在编译阶段编译器无法确定其具体的类型。因此会产生逃逸，最终分配到堆上