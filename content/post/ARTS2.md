---
title: "ARTS #2"
date: 2021-07-07T20:34:39+08:00
---

## Algorithm
 [83\. 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)
非常简单的单向链表遍历
```
# Definition for singly-linked list.
# class ListNode(object):
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution(object):
    def deleteDuplicates(self, head):
        """
        :type head: ListNode
        :rtype: ListNode
        """
        if head is None:
            return head
        start = head
        current_node = head
        next_node = None
        while current_node.next:
            next_node = current_node.next
            if current_node.val != next_node.val:
                current_node = next_node
            if current_node.val == next_node.val:
                if next_node.next:
                    current_node.next = next_node.next
                else:
                    current_node.next = None
        return start
```

## Review
[Easy functional programming techniques in Go](https://deepu.tech/functional-programming-in-go/)
#### 函数编程需要尽量做到的几个概念
- 高阶函数：assign functions to variables, pass a function as an argument to another function or return a function from another
- 闭包：从函数外部传值进入函数，在函数内部使用 并且在离开外部变量作用域后仍然可以使用该变量
- 纯函数： a pure function should return values only based on the arguments passed and should not affect or depend on global state
- 递归：Functional programming favors recursion over looping
- 惰性计算：Lazy evaluation or non-strict evaluation is the process of delaying evaluation of an expression until it is needed。 spark就是典型的lazy evaluation
- Referential transparency：Functional programs do not have assignment statements, that is, the value of a variable in a functional program never changes once defined。go中默认是传值引用，因此可以实现referential transparency，但是注意不要传指针 不然就会破坏referential transparency.
#### 个人观点
通过这篇文章对函数编程有了更清晰的理解，之前对函数式编程就是简单的认为有了function first和闭包就是函数式编程。其实函数式编程应该是一个编程范式（或者说编程习惯），本质应该是像数学公式一样：数学公式只会通过输入的内容进行计算然后返回结果，而不会去改变输入值或者外部其他变量。 function first、闭包、lambda是实现函数式编程范式的一些手段

另附上 [robpike](https://github.com/robpike)大神用golang实现的几个高阶函数： [golang高阶函数](https://github.com/robpike/filter) 虽然大神说不建议使用，应该使用for循环😂

## Tip
项目开发中遇到一个需求：
- 对外提供一个查询接口A，调用方调用我这个接口进行信息查询，入参为一个list
- 我需要根据这个list的内容调用第三方服务X进行内容查询 同时服务X的做了限速，查询qps只能为5
- 需要将list的查询结果做聚合，再异步回调给调用者
针对以上需求，设计了一个方案:
1. 全局设计一个任务队列 来的请求都创建一个任务往队列中丢 同一个请求创建的任务，都存一个resultChannel的指针。
2. 如果任务队列中有任务同时定时器达到200ms定时时刻，则起一个协程进行调用服务X的请求，请求结果回写到resultChannel中。
3. 通过waitgroup判断同一个请求的所有任务是否完成，如果都完成则从resultChannel中读取结果并进行聚合操作。
知识点：waitgroup的使用、channel中套channnel、时间队列的使用（下周的ARTS认真看下timmer的源码然后分享下）
```
var Timer = make(chan time.Time)
var TaskPool = make(chan *Task)

func producerFunc(taskName string, burstyTasks chan *Task) {
	var wg sync.WaitGroup
	var resultChannel = make(chan string, 10)
	for i := 0; i < 10; i++ {
		wg.Add(1)
		i := i
		go func() {
			result := producer(i, taskName)
			burstyTasks <- &Task{result, &wg, resultChannel}
		}()
	}
	wg.Wait()
	for i := 0; i < 10; i++ {
		fmt.Println(<-resultChannel)
	}
}

func producer(i int, taskName string) string {
	time.Sleep(10 * time.Millisecond)
	return taskName + " and " + strconv.Itoa(i)
}

func consumer(taskPara *Task) string {
	result := "result of " + taskPara.producerResult
	return result
}

func rateLimitFunc(burstyLimiter chan time.Time) {
	for t := range time.Tick(400 * time.Millisecond) {
		burstyLimiter <- t
	}
}

type Task struct {
	producerResult string
	Wg             *sync.WaitGroup
	ResultChannel  chan string
}

func RateLimit() {
	go func() {
		rateLimitFunc(Timer)
	}()

	go func() {
		for task := range TaskPool {
			task1 := task
			<-Timer
			go func(taskPara *Task) {
				time.Sleep(10 * time.Millisecond)
				result := consumer(taskPara)
				task.ResultChannel <- result
				fmt.Println("task:", result, time.Now())
				taskPara.Wg.Done()
			}(task1)
		}
	}()

	go func() {
		producerFunc("task1", TaskPool)
	}()

	go func() {
		producerFunc("task2", TaskPool)
	}()

	time.Sleep(200 * time.Second)
}
```
## Share
[Product Integration Testing at the Speed of Netflix](https://netflixtechblog.com/product-integration-testing-at-the-speed-of-netflix-72e4117734a7)
这片文章介绍了netfix如何如何进行端到端质量保障
1. 对HIT（High Impact Titles）的质量保障
  - 发布前：因为需求可能会变动，所以以手工测试为主
  - 发布后：自动化为主 我理解就是进行用例和功能回归
2. A/B测试自动化
其实和A/B测试没啥关系，主要就是说了以下点：
  - 使用脚本进行自动化。通过让脚本足够参数化、封装，最终让脚本可reused。这样子后续的重复开发成本比较低
  - 使用python进行自动化开发、同时配合jenkins流水线来控制自动化执行流程
3. 国际化适配
  - 在步骤2的脚本代码基础上，使用opt-in model设计模式封装然后简单的每个国家跑一遍一样的脚本
  - 因为需要重复的跑不同国家的自动化脚本，所以引入jenkins流水线的并发 来提高执行效率

整篇文章看下来，可以发现没什么比较有借鉴性的地方，基本和自己之前做的自动化差不多 文中有一句话给其他想入手自动化测试的QA同学借鉴：treat test automation as a deliverable product and focus on delivering a minimum viable product (MVP) composed of reusable pieces. 就是自动化脚本其实也是代码，也要考虑可重用性、组件封装