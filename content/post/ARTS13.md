---
title: "ARTS13"
date: 2021-08-29T16:13:34+08:00
---

## Algorithm
#### [剑指 Offer 10- II. 青蛙跳台阶问题](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)
简单动态规划题目，注意边界值和取模即可
```
func numWays(n int) int {
	if n < 2 {
		return 1
	}
	result := make([]int, n+1)
	result[0] = 1
	result[1] = 1
	index := 2
	for index <= n {
		result[index] = result[index-1] + result[index-2]
		result[index] = result[index] % 1000000007
		index++
	}
	return result[n]
}
```
## Review
[Go: Understand the Empty Interface](https://medium.com/a-journey-with-go/go-understand-the-empty-interface-2d9fc1e5ec72)
文章介绍了Golang的interface{}的底层实现：
1. Golang的runtime源码中，有一个专门的emptyInterface的结构体来表示interface{}。其实就是2个指针：一个代表这个interface的内置type，另一个则是存具体数据内容的指针。前者里面存了内置type具体的size、kind等内容。
```
type emptyInterface struct {
   typ  *rtype            // word 1 with type description
   word unsafe.Pointer    // word 2 with the value
}
type rtype struct {
   size       uintptr
   ptrdata    uintptr
   hash       uint32
   tflag      tflag
   align      uint8
   fieldAlign uint8
   kind       uint8
   alg        *typeAlg
   gcdata     *byte
   str        nameOff
   ptrToThis  typeOff
}
```
2. 而一个具体的struct则包含一下内容，所以我们在进行interface转具体struct的时候，就需要把这2个struct的属性一个一个进行映射，这其中的性能消耗是很可观的。
```
type structType struct {
   rtype
   pkgPath name
   fields  []structField
}
```
3. 做了一个类型转换的性能比对。interface{}转换为具体struct性能消耗很大。
4. 文章最后给了一个建议，如果确实形参数要传inerface{}，然后做类型转换。建议传指针，然后对指针进行类型转换。
## Tip
之前使用了[koro1FileHeader](https://github.com/OBKoro1/koro1FileHeader)插件，能帮助我们自动添加文件注释。但是有一个问题就是这个插件会自动帮所有文件加上注释，有时候一些代码项目中一些不需要添加注释的文件也会加上（比如json、yaml、md等）。今天看了官方文档，可以在setting.json中配置上exclude，这样子这些文件就不会自动加上注释了
```
"fileheader.configObj": {
        "autoAdd": true, // 自动添加头部注释开启才能自动添加
        "autoAlready": false, // 默认开启
        "prohibitAutoAdd": [
            "json",
            "yaml",
            "md"
          ],
    },
```
## Share
[生成订单30分钟未支付，则自动取消，该怎么实现？](https://mp.weixin.qq.com/s/-fmKcw2m2eb6NRAmcXfBhw)

这周在做的一个需求，其中有部分是：在一定时间之后，解散机器人自动创建的对应聊天群。刚好看到了这篇文章，下周试一下用文中介绍的方法。