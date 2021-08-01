---
title: "ARTS9"
date: 2021-08-01T22:02:59+08:00
---

## Algorithm
#### [剑指 Offer 25\. 合并两个排序的链表](https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof/)
解法1：起一个新的链表。然后双指针遍历2个链表，不断地new新的ListNode，并添加到新的链表中。
解法2：三指针解法。将2个链表合并为一个结果链表。
  - 指针A指向合并后的链表的当前位置。
  - 另外2个指针B、C指向未合并前的2个链表的Next节点位置。当其中一个指针达到尾部之后，将指针A的Next指向另一个指针即可。
```
package merge_two_lists

//Definition for singly-linked list.
type ListNode struct {
	Val  int
	Next *ListNode
}

func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
	if l1 == nil && l2 == nil {
		return nil
	}
	if l1 == nil {
		return l2
	}
	if l2 == nil {
		return l1
	}
	var result, current *ListNode
	if l1.Val <= l2.Val {
		current = l1
		l1 = l1.Next
	} else {
		current = l2
		l2 = l2.Next
	}
	result = current
	for l1 != nil && l2 != nil {
		if l1.Val < l2.Val {
			current.Next = l1
			current = current.Next
			l1 = l1.Next
			continue
		} else {
			current.Next = l2
			current = current.Next
			l2 = l2.Next
			continue
		}
	}
	if l1 != nil {
		current.Next = l1
	} else {
		current.Next = l2
	}
	return result
}
```
## Review
[How I got a job at GitLab!](https://rajendraak.medium.com/how-i-got-a-job-at-gitlab-a3515214b74b)
文章是一个哥们讲述自己如何通过努力达成自己的目标：获得gitlab公司的offer。主要有以下几点：
1. 为自己定制目标，并在public上公布出来。以此做为动力或者督促。
2. 在达到目标的过程主动寻求他人的目标。
3. 坚持下去，并且享受这个过程。

文章最后有一句话：Be consistent, work on something with conviction and persistence and the results will get to you on their own!
兴趣就是最好的老师。享受达到目标的过程很重要。大学的时候虽然是软件工程专业，但是对写代码没有一点兴趣，所以混了4年，毕业了连个hello world都写不好。反倒是毕业4，5年后的最近，开始慢慢get到编程的乐趣和成就感，所以愿意在工作之余花时间去时间学习和提升编程能力。
最后：Whatever you do, enjoy the journey, the destination will come to you, you don’t have to move anywhere!
## Tip
这周学到了一个新的单元测试技巧，mock方法。Golang function mock
```
type PageGetter func(url string) string

func downloader(pageGetterFunc PageGetter) {
    // ...
    content := pageGetterFunc(BASE_URL)
    // ...
}

func mock_get_page(url string) string {
    // mock your 'get_page()' function here
}

func TestDownloader(t *testing.T) {
    downloader(mock_get_page)
}
```
来源：[Mock functions in Go](https://stackoverflow.com/questions/19167970/mock-functions-in-go)
## Share
分享这周写的Golang可测性改造总结
[Golang单元测试实战二：依赖注入与面向接口改造](https://www.jianshu.com/p/8ea2c7a643bb)