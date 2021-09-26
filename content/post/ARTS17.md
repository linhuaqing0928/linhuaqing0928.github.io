---
title: "ARTS17"
date: 2021-09-26T22:15:29+08:00
---

## Algorithm
#### [剑指 Offer 29\. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)
```
func spiralOrder(matrix [][]int) []int {
	if len(matrix) == 0 {
		return nil
	}
	result := make([]int, 0, len(matrix)*len(matrix[0]))
	xMin := 0
	yMin := 1 // 这里需要注意，Y轴刚起步就占用了一个，所以初始为1
	yMax := len(matrix)
	xMax := len(matrix[0])
	x := 0       // 代表X轴
	y := 0       // 代表Y轴
	flag := true // true代表横着走，false代表竖着走
	count := 1   // 1代表正向走，-1达标逆向走
	for {
		if flag {
			if x < xMin || x >= xMax {
				break
			}
		} else {
			if y < yMin || y >= yMax {
				break
			}
		}
		result = append(result, matrix[y][x])
		// 专项判断
		if flag {
			xTemp := x + count
			if xTemp >= xMax {
				flag = false
				count = 1
				xMax--
			} else if xTemp < xMin {
				flag = false
				count = -1
				xMin++
			}
		} else if !flag {
			yTemp := y + count
			if yTemp >= yMax {
				flag = true
				count = -1
				yMax--
			} else if yTemp < yMin {
				flag = true
				count = 1
				yMin++
			}
		}
		// 正式开始转向
		if flag {
			x += count
		} else {
			y += count
		}
	}
	return result
}
```
## Review
[Decision Making at Netflix](https://netflixtechblog.com/decision-making-at-netflix-33065fa06481)
[What is an A/B Test?](https://netflixtechblog.com/what-is-an-a-b-test-b08cc1b57962)
netflix关于A/B实验的系列文章，从为什么需要到什么是A/B实验，讲得挺不错。

文章开头讲到一般公司做decision不外乎几种方式：
- Let leadership make all the decisions.
- Hire some experts in design, product management, UX, streaming delivery, and other disciplines — and then go with their best ideas.
- Have an internal debate and let the viewpoints of our most charismatic colleagues carry the day.
- Copy the competition

之前在稿定做视频剪辑的时候，感觉就是这样子的。先是竞品有啥比较受欢迎的功能，咱也得做。然后实际做的过程就是组内几个产品和交互同学根据经验设计出几个方案，最后给老板拍板做决定。没有A/B实验和灰度啥的（当然很大一个因素也是因为当时稿定的基础设施建设较差，没有进行A/B的能力）

来了字节以后感觉还是相对专业点，所有需求上线前都会先进行A/B实验，然后根据相关指标数据做decision。 
## Tip
这周在做压测数据构造的时候，发现公司内部有一个自动生成RPC调用client的工具Overpass，省了很多繁琐的thrift_gen工作。
## Share
[MySQL执行顺序](https://mp.weixin.qq.com/s/sdFXpGntlL4KmDEDSLrR1A)
分享一下最近看到的mysql执行顺序的小文章