---
title: "ARTS26"
date: 2021-12-19T22:08:36+08:00
---

## Algorithm
#### [剑指 Offer 42\. 连续子数组的最大和](https://leetcode-cn.com/problems/lian-xu-zi-shu-zu-de-zui-da-he-lcof/)
动态规划解法，指针往前移动的话，需要在自己本身和自己本身+上一步最大值之和中比较。
转移公式为：f(i)=max{f(i−1)+nums[i],nums[i]}
```
// 动态规划
func maxSubArray1(nums []int) int {
	results := make([]int, len(nums)) // 用来存储每个节点上的最大值
	results[0] = nums[0]
	max := nums[0]
	current := 1
	for current < len(nums) {
		// 动态规则选择往前走一步最大值是自己本身还是自己本身和上一步最大值之和
		if results[current-1]+nums[current] > nums[current] {
			results[current] = results[current-1] + nums[current]
		} else {
			results[current] = nums[current]
		}
		// 顺便取最大值
		if results[current] > max {
			max = results[current]
		}
		current += 1
	}
	return max
}

// 基于动态规划进一步优化空间复杂度成O（1）
func maxSubArray(nums []int) int {
	current := 1
	max := nums[0]
	pre := nums[0]
	for current < len(nums) {
		if nums[current]+pre > nums[current] {
			pre = nums[current] + pre
		} else {
			pre = nums[current]
		}
		current += 1
		if pre > max {
			max = pre
		}
	}
	return max
}
```
## Review
[SOLID Principles in Golang Explained with Examples](https://levelup.gitconnected.com/solid-principles-in-golang-explained-by-examples-4a4cccf47388)
文章举了个例子，如何重构一个方法让他更加符合单一职责原则。翻了下自己最近写的业务代码，一个service的Public方法就有230行代码，包含了调接口1、调接口2、调接口3、数据整合、调Dao层入DB，并且包含各种判空代码。距离单一职责还差得很远，后面还是需要多多注意，养成代码“洁癖”。
## TIP
这周做业务过程，用到了Go sql.Rows的scan方法。踩了一个之前开发同学遗留的坑。

如果Rows里面有null值，scan是会报错的，但是之前的历史代码没有对这个error进行判断而是默认全部成功，进而引发一系列报错。

解决办法就是在初始化destination指针的时候，如果数据库对应field是nullable的话，可以声明destination为对应类型的sql.NullInt64或者sql.NullString等。然后使用XXX.Valid来判断这个值是Null还是确实是0或者“”值。
## Share
这周开始跟着B站“操作系统32讲”来学习复习操作系统。
### L1 什么是操作系统：
1. 操作系统主要职责：CPU管理、内存管理、终端管理、磁盘管理、文件管理、网络管理、电源管理、多核管理。
### L2 揭开钢琴的盖子
#### 图灵机
图灵机的组成：控制器、纸带、读写头。

图灵机模拟你在纸上计算3+2=5：图灵机的控制器代表人的大脑，纸带代表人手上的纸，读头代表人的眼睛，写头代表笔。一开始纸带上有3、2、+，读头获取到这些内容，然后控制器差表得到结果5，再通过写头把5写到纸带上。这样子图灵机就完成了3+2=5的计算模拟过程。 

上面描述的过程，图灵机的控制器会计算3+2是因为事先在控制器内有一张写死的表（或者逻辑），里面明确3+2是等于5。如果换成3*2的话，这时候就需要修改控制器的逻辑。也就是说这个图灵机是特定图灵机（只会计算特定逻辑）而不是通用图灵机。
#### 通用图灵机
将图灵机的+（算法）的逻辑挪到纸带上，控制器的职责变成解析纸带上内容然后在控制器上生成新的算法逻辑。这样子就变成了图灵机就变成通用了，使用者不用更换控制器，只需要更改纸带上的算法逻辑就可以实现程序计算。
#### 冯诺依曼机
冯诺依曼机五大部件：输入设备、输出设备、存储器、运算器、控制器。
和通用图灵机相比，引入了存储程序（内存）的思想。也就是上诉的纸带拆分成了：输入设备、输出设备、存储器。

之前的纸带同时负责算法逻辑的存储（+-*/）、输入数据数据（3、2）、计算结果（5）的呈现，控制器从纸带上载入算法逻辑。

冯诺依曼机则是让程序员事先将之前放在纸带上的算法逻辑（指令）写到存储设备上，然后控制器（CPU）再从存储设备中载入这些指令进行解释执行，最后进行算法执行。这整个过程叫做：取指执行
#### 打开电源发生了什么（X86 PC）
名词：BIOS：basic input ouput system 基本输入输出系统
实模式的寻址规则：CS：IP（CS左移4个位置+IP）
1. 刚开机时候CPU处于实模式
1. 一开始CS=0xFFFF；IP=0x0000（写死）
1. CPU寻址到0xFFFF0（也就是ROM BIOS映射区，这个是固化在硬件里面的）
1. 检查RAM、磁盘、显示器、软硬磁盘。
1. 将磁盘的0磁道0扇区（引导扇区）内容读入然后放到0x7c00处
1. 然后设置cs=0x07c0，ip=0x0000，也即是下一次CPU会去引导扇区进行后续的取指执行操作。
1. CPU再使用汇编依次读取boot扇区、setup扇区、system模型（os代码）载入到内存里面。