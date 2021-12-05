---
title: "ARTS24"
date: 2021-12-06T00:41:46+08:00
---

## Algorithm
#### [剑指 Offer 18\. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)
```
//   Definition for singly-linked list.
type ListNode struct {
	Val  int
	Next *ListNode
}

func deleteNode(head *ListNode, val int) *ListNode {
	if head.Val == val {
		return head.Next
	}
	result := head
	for {
		if head.Next == nil {
			break
		}
		if head.Next.Val == val {
			head.Next = head.Next.Next
			break
		}
		head = head.Next
	}
	return result
}
```
## Review
[When Do Programmers Retire? Is 35 the End?](https://betterprogramming.pub/when-do-programmers-retire-is-35-the-end-72d173760ee2)

程序员35岁真的会被裁吗？文章讲述了几个导致现在市面上这一主流观点形成的原因，最终阐述了作者关于怎么避免这一情况发生的的观点：Be updated all the time。

文章里面有一个数据我觉得很值得探讨：亚洲国家（特别是是印度）程序员的平均年龄比美国、澳大利亚低很多。

为什么呢？我认为是因为亚洲国家（包括中国目前的很多互联网公司）的软件开发人员目前大部分还是处于“劳动力密集型”阶段，很多程序员做的事情并没有很高深。随便找个刚毕业的大学生培训一下跟几个项目之后，产出就能和工作好几年的程序员持平。如果是这样的话，企业为什么还要花大价钱招聘资深程序员呢？

那么怎么摆脱这个困境呢？Be updated all the time，保持技术深度和新鲜度，体现出你和新人的差距，这样子老板才会愿意花大价钱雇你。
## TIP
这周在工作过程中用上了之前网上看到的go里面封装事务传递的方法。还犯了一个低级错误：开启事务之后没有手动commit，结果一直没有正式提交到数据库，花了一早上的时间排查这个问题。

挺好的，踩坑才能让自己对知识点印象更深刻。
```
func Tx(tx *gorm.DB, funcs ...func(db *gorm.DB) error) (err error) {
	tx = tx.Begin()
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
## Share
[Redis布隆过滤器原理与实践](https://juejin.cn/post/6917802871341711367)

工作过程中遇到了布隆过滤器，所以找文章简单了解了下。简单来说就是：
1. 首先存在一个布隆过滤器存储位数组A。
1. 每次要保存的一个值，则将这个值通过几个hash公式来计算输入值得到一个位结果，然后将这个结果存在位数组A中。
1. 如果要查询一个值是否存在，通过同样的hash公示计算出这个值的结果，然后和已经存在的位数组对比。如果都为0则可以确认是这个要查询的值不在已经保存的值范围内，如果存在一个值为1则代表要查询的值可能在已经保存的值范围内。


1. 优先：优点很明显，二进制组成的数组，占用内存极少，并且插入和查询速度都足够快
缺点
1. 缺点：随着数据的增加，误判率会增加；还有无法判断数据一定存在；另外还有一个重要缺点，无法删除数据

可以用来做数据量比较大的去重工作。