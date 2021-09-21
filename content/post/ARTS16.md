---
title: "ARTS16"
date: 2021-09-21T22:46:04+08:00
---

## Algorithm
#### [剑指 Offer 15\. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)
1. 自己的解法：不断的除和求余
2. 官方解法：与运算
```
func hammingWeight1(num uint32) int {
	count := 0
	index := 31
	for index >= 0 {
		multi := num / (uint32(math.Pow(2, float64(index))))
		if multi == 1 {
			count++
		}
		num = num % (uint32(math.Pow(2, float64(index))))
		index--
	}
	return count
}

func hammingWeight2(num uint32) int {
	count := 0
	index := 0
	for index < 32 {
		if 1<<index&num > 0 {
			count++
		}
		index++
	}
	return count
}

```
## Review
[Golang: Six Error Handling techniques to help you write elegant code](https://medium.com/higher-order-functions/golang-six-error-handling-techniques-to-help-you-write-elegant-code-8e6363e6d2b)
文章比较细，介绍了作者个人比较推荐的几个error处理方式。感觉比较有用的2点：
1. 遇到error直接return 而不是多层嵌套。这个我是比较赞同的，同时自己个人编码过程也是习惯这样做的。
2. 推荐了一个封装了退避算法的重试库[backoff](https://github.com/cenkalti/backoff)。 感觉挺有用的后面工作过程可以引入用一下，现在都是自己比较low的手动retry 3次。
## Tip
get到了[backoff](https://github.com/cenkalti/backoff)，好用的退避重试库。
## Share
[深度解密 Go 语言之 sync.Pool](https://www.cnblogs.com/qcrao-2018/p/12736031.html)
分享一个sync.Pool的文章。自己尝试读了第一遍，读到“源码分析”部分的时候有点卡壳。先mark，过几天有完整的时间再重新啃一遍。