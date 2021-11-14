---
title: "ARTS21"
date: 2021-11-14T23:02:34+08:00
---

## Algorithm
#### [剑指 Offer 22\. 链表中倒数第k个节点](https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof/)
1. 最简单的方法，遍历链表 然后每个节点存储在map里面，最后按照key取出来倒数第K个节点
2. 快慢指针方法 快指针比慢指针早出发K-1个节点 这样子当快指针达到尾节点的时候，慢指针就是我们期望的结果
```
// map存储法
func getKthFromEnd(head *ListNode, k int) *ListNode {
	if head == nil {
		return nil
	}
	nodes := make(map[int]*ListNode)
	count := 1
	for {
		nodes[count] = head
		if head.Next != nil {
			head = head.Next
			count++
		} else {
			break
		}
	}
	result := nodes[count-k+1]
	return result
}

// 快慢指针方法
func getKthFromEnd1(head *ListNode, k int) *ListNode {
	if head == nil {
		return nil
	}
	slow := head
	fast := head
	index := 1
	for {
		if fast.Next != nil {
			if index < k {
				fast = fast.Next
				index++
			} else {
				fast = fast.Next
				slow = slow.Next
			}
		} else {
			break
		}
	}
	return slow
}

```
## Review
[Practical DDD in Golang: Value Object](https://levelup.gitconnected.com/practical-ddd-in-golang-value-object-4fc97bcad70)
文章讲了DDD在golang中的几个实践，截取几个个人认为比较有用的建议：
1. 唯一性和equal方法：entity和value object的一个关键性的差别就在于value object没有唯一标识uuid，那么怎么比较2个相同struct的value object呢？文章里面介绍的就是实现一个EqualTo方法。
1. value object另一个特性是需要具有不变性。在go中实现这个特性的建议实践就是，不要提供指针方法而是结构体方法。如果这个方法的返回值需要一个经过处理的value object的时候，new一个实例然后返回，而不是对老实例的指针进行操作。
## TIP
rocketMQ开启了手动ACK之后，如果在消费者端发生了消息丢失的话，rocketMQ的故障转移特性会保障其他消费者能继续消费这个丢失的消息。具体场景如下：
1. 消费者A从MQ中消费到event 1.
2. 消费者A的handler处理event1的过程，如果A挂了。这时候消费者A和MQ之间的链接会断开。
3. MQ感知到消费者A挂了，启动故障转移特性，会将event1重新推送给其他消费者。
4. 其他消费者消费处理event1。
## Share
rabbitMQ消息丢失的几个场景以及解决方案：
1. 在生产者发送消息之后（可能是网络问题导致消息在到达MQ之前丢失了）发生消息丢失。这时候可以通过以下2种方案解决：
  - 开启MQ的事务功能。在生产者发送消息之前开始事物，然后发送消息 如果MQ没有及时接收到消息，这时候生产者会收到异常报错，这时候生产者就可以根据异常报错进行发送消息的重试。如果没有接到报错，就提交事务commit。这里是同步堵塞机制，性能比较差。
  - 开启comfirm模式。在生产者那里设置开启 confirm 模式之后，你每次写的消息都会分配一个唯一的 id，然后如果写入了 RabbitMQ 中，RabbitMQ 会给你回传一个 ack 消息，告诉你说这个消息 ok 了。如果 RabbitMQ 没能处理这个消息，会回调你的一个 nack 接口，告诉你这个消息接收失败，你可以重试。comfirm是异步机制，性能比较好。
2. MQ丢失了消息（MQ挂了）。这种场景需要开启MQ的持久化功能，这样子MQ挂了重启之后也可以通过重新读区磁盘来获取之前的消息内容。
3. 消费者端消息丢失。这种场景需要关闭消费者端的自动ACK，每次消费者消费完event之后手动ack。这样子如果消费者在消费过程挂了，MQ没有接收到ack就会把消费分配给其他消费者去处理，从而保障不会消息丢失。

参考：[消息中间件面试题：消息丢失怎么办？](https://www.jianshu.com/p/4491cba335d1)