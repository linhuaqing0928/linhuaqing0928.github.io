---
title: "ARTS25"
date: 2021-12-13T00:25:20+08:00
---

## Algorithm
#### [剑指 Offer 26\. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)
2层嵌套dfs，外层dfs用来深度遍历以A的节点为root的子树，内层dfs用来遍历判断当前A的节点为root的子树和B是否一致。
```
package is_sub_structure

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func isSubStructure(A *TreeNode, B *TreeNode) bool {
	if B == nil {
		return false
	}
	if dfs(A, B) {
		return true
	}
	if A.Left != nil {
		if isSubStructure(A.Left, B) {
			return true
		}
	}
	if A.Right != nil {
		if isSubStructure(A.Right, B) {
			return true
		}
	}
	return false
}

func dfs(A *TreeNode, B *TreeNode) bool {
	if A.Val == B.Val && B.Left == nil && B.Right == nil {
		return true
	}
	if A.Val != B.Val {
		return false
	}
	if B.Left != nil && A.Left == nil {
		return false
	}
	if B.Right != nil && A.Right == nil {
		return false
	}
	if A.Left != nil && B.Left != nil {
		if !dfs(A.Left, B.Left) {
			return false
		}
	}
	if A.Right != nil && B.Right != nil {
		if !dfs(A.Right, B.Right) {
			return false
		}
	}
	return true
}
```
## Review
[The Biggest Lie in Software Engineering](https://medium.com/codex/the-biggest-lie-in-software-engineering-f0516d147106)
这篇文章讨论了一个很有意思的话题，那就是如果在面试过程中面试官问你：How do you keep up with industry trends?  该怎么回答？

我在之前的面试中也问过这个问题，可以很明显的发现大部分面试的同学其实并没有在这方面花时间，但是为了给面试官留下好印象就是“编”一些学习习惯。大家最经常“编“的就是：读公众号文章、看技术视频学习，如果再进一步问细节就会发现答不上来。这样子往往会给面试官留下更不好的印象。

可能会有同学说了，你这不是”卷“吗？平时工作那么忙，下班不是应该休息嘛？哪里来的时间再去学习新知识。 emm，有一定道理。

说说我的个人观点：
1. 诚实，这个最关键 如实回答没有经常学习的习惯比回答有然后被人拆穿好多了
1. 学习和进步应该和工作挂钩。像我个人习惯就是，工作中遇到什么问题、新知识点记录下来，然后再延伸去进一步系统性的、更深度的学习。坚持下去，工作中就不会是大家口中的crud boy了。
1. 好记性不如烂笔头，养成写技术文章的习惯，方便后续回顾。面试的时候把你的学习记录show给面试官，相信会加不少分。
1. enjoy coding。毕竟要工作3，40年呢。
## TIP
这周在工作过程学会了使用Go标准库的encoding/csv来读取和写入csv文件。
csv.NewReader(XXX)打开文件句柄，然后使用ReadAll()和Read()能够读取到csv文件内容。每个\n会解析成一个string数组，而每个,分割出来的string是这个数组的一个element。

使用csv.NewWriter(XXX)能够往csv文件写入内容。调用Write(record []string)然后Flush()，即可往csv文件中写入对应内容。也可以WriteAll(records [][]string)
## Share
[IO模型：同步、异步、阻塞、非阻塞](https://blog.csdn.net/lisonglisonglisong/article/details/51944671?spm=1001.2101.3001.6650.8&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBLOGCOLUMN%7Edefault-8.highlightwordscore&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBLOGCOLUMN%7Edefault-8.highlightwordscore)
几种IO模型的差别，脑海里面先有个概念 后续再进一步深入学习