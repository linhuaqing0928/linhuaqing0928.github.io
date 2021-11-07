---
title: "ARTS20"
date: 2021-11-07T22:51:24+08:00
---

## Algorithm
#### [剑指 Offer 28\. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)
```
//   Definition for a binary tree node.
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func isSymmetric(root *TreeNode) bool {
	if root == nil {
		return true
	}
	return symmetric([]*TreeNode{root.Left, root.Right})
}

func symmetric(roots []*TreeNode) bool {
	length := len(roots)
	index := 0
	for index < length/2 {
		temp1 := roots[index]
		temp2 := roots[length-1-index]
		if temp1 != nil && temp2 != nil && temp1.Val != temp2.Val {
			return false
		}
		if (temp1 != nil && temp2 == nil) || (temp1 == nil && temp2 != nil) {
			return false
		}
		index++
	}
	empty := true
	nextRoots := []*TreeNode{}
	index = 0
	for index < length {
		if roots[index] == nil {
			nextRoots = append(nextRoots, nil)
			nextRoots = append(nextRoots, nil)
			index++
			continue
		}
		nextRoots = append(nextRoots, roots[index].Left)
		if roots[index].Left != nil {
			empty = false
		}
		nextRoots = append(nextRoots, roots[index].Right)
		if roots[index].Right != nil {
			empty = false
		}
		index++
	}
	if !empty {
		return symmetric(nextRoots)
	}
	return true
}
// unit test
func TestIsSymmetric(t *testing.T) {
	input := []int{1, 2, 2, 3, 4, 4, 3}
	root := buildTree(input)
	// fmt.Println(root)
	result1 := isSymmetric(root)
	if result1 != true {
		t.Errorf("TestIsSymmetric failed, got %v, want %v", result1, true)
	}
}

func buildTree(input []int) *TreeNode {
	root := &TreeNode{input[0], nil, nil}
	lastNodes := []*TreeNode{root}
	nextNodes := []*TreeNode{}
	level := 1
	start := 1
	end := start + int(math.Pow(2, float64(level)))
	for end <= len(input) {
		for index, node := range lastNodes {
			index := index
			node := node
			if node == nil {
				continue
			} else {
				first := input[start+2*index]
				if first != -1 {
					firstNode := &TreeNode{first, nil, nil}
					node.Left = firstNode
					nextNodes = append(nextNodes, firstNode)
				} else {
					nextNodes = append(nextNodes, nil)
				}
				second := input[start+2*index+1]
				if second != -1 {
					secondNode := &TreeNode{second, nil, nil}
					node.Right = secondNode
					nextNodes = append(nextNodes, secondNode)
				} else {
					nextNodes = append(nextNodes, nil)
				}
			}
		}
		lastNodes = nextNodes
		nextNodes = []*TreeNode{}
		start = end
		level++
		end = start + int(math.Pow(2, float64(level)))
	}
	return root
}
```
## Review
[Why goroutines are not lightweight threads?](https://codeburst.io/why-goroutines-are-not-lightweight-threads-7c460c1f155f)
文章有点标题党，但是大概比较了go的协程和操作系统的线程：
1. 线程是OS层面，Go的协程则是应用程序层面
1. 协程内存开销比线程小，只需要2KB
1. 协程切换比线程切换需要恢复的寄存器少（only 3 registers i.e. PC, SP and DX (Data Registers) being updated during context switch rather than all registers (e.g. AVX, Floating Point, MMX)）
1. 协程本质是运行在线程上的，由go runtime来统一调度goroutine在多个线程之间运行和切换，详细可以了解GMP
## TIP
使用gorm如果涉及到跨表操作，不同dao之间怎么更优雅的实现事务传递。封装一个TX方法
```
func Tx(funcs ...func(db *gorm.DB) error) (err error) {
    tx := DB.Begin()
    defer func() {
        if r := recover(); r != nil {
            tx.Rollback()
            err = fmt.Errorf("%v", err)
        }
    }()
    for _, f := range funcs {
        err = f(tx)
        if err != nil {
            tx.Rollback()
            return
        }
    }
    err = tx.Commit().Error
    return
}
```
来源：https://www.jianshu.com/p/a84b3137fdb9
## Share
双11期间压测进行遇到数据库分片热点问题，之前对数据库只有一个模糊的概念。今天学习一下数据库分片的基础入门知识。

数据库分片的好处：
1. 方便数据库服务水平扩容
1. 减少查询响应时间
1. 减少数据库服务宕机造成的影响（分布式系统的好处）

数据库分片的缺点：
1. 因为需要在应用层进行数据分片，增加了程序整体的复杂度（读与写都需要考虑）。
1. 如果数据库分片hash算法不均匀，可能导致热点问题，需要重新分片。这也是我们压测数据构造经常会遇到的问题。
1. 一旦进行分片，数据要再恢复到未分片的架构成本就会很高。

主要的分片架构：
1. Key Based Sharding 基于键的分片：指定数据表中的一列作为分片键，然后通过一个固定的hash算法来对同一个表中的数据进行分片。
2. Range Based Sharding 基于范围的分片：简单的指定键值范围，然后基于给定值的范围进行数据分片。
3. Directory Based Sharding 基于目录的分片：维护一个查找表，该查找表使用分片键来跟踪哪个分片包含哪些数据

内容来源：[数据库分片（Database Sharding)详解](https://zhuanlan.zhihu.com/p/57185574)