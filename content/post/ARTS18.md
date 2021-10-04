---
title: "ARTS18"
date: 2021-10-04T23:24:55+08:00
---

## Algorithm
#### [剑指 Offer 14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)
2种解法：
1. 贪心法，从长度为2的绳子开始找规律 可以知道每次得到越多的3的时候 乘积越大。这个解法是自己解出来的。
2. 动态规划，看其他人借题答案学到的。关键点动态规划点就是每次先剪去一段长度，然后剩下的有剪或者不剪2种方法，取max。转移方程为：dp[i] = max(dp[i], max(j * (i - j), j * dp[i - j]))
```
// 贪心算法 找规律，可以得到将内容划分为3的时候 得到的乘积为最大
func cuttingRope1(n int) int {
	var result int
	if n == 2 {
		return 1
	}
	if n == 3 {
		return 2
	}
	remind := n % 3
	if remind == 0 {
		result = int(math.Pow(3, float64(n/3)))
	}
	if remind == 1 {
		result = int(math.Pow(3, float64(n/3)-1)) * 4
	}
	if remind == 2 {
		result = int(math.Pow(3, float64(n/3))) * 2
	}
	return result
}

// 动态规划
// 因为必须m>1，所以必须剪一次 假设剪掉的一段长度为J，则剩余的长度为n-j 剩余的绳子可以有2种选择：剪或者不剪 剪的话，就按照之前最优解的乘积*j 不剪的话，就是（n-j）*j 取max得出最优解
// J的长度是变化的，但是至少从2开始 这里也要取max
func cuttingRope(n int) int {
	dp := make([]int, n+1)
	dp[2] = 1
	i := 3
	for i <= n {
		j := 2
		for j < i {
			dp[i] = int(math.Max(float64(dp[i]), math.Max(float64(j*(i-j)), float64(j*(dp[i-j])))))
			j++
		}
		i++
	}
	return dp[n]
}

```
## Review
[Go concurrency pattern: Pipeline](https://syafdia.medium.com/go-concurrency-pattern-pipeline-635dfae01af1)
[Go concurrency pattern: Semaphore](https://levelup.gitconnected.com/go-concurrency-pattern-semaphore-9587d45f058d)
[Go Concurrency Pattern: Worker Pool](https://medium.com/code-chasm/go-concurrency-pattern-worker-pool-a437117025b1)
Golang本身标准库没有信号量、线程池等内置功能，这几篇文章分别介绍了如何在golang中实现这几种并发模式。
1. pipline和模板设计模式有点像，上一个步骤的output是下一个步骤的input。通过一个类型为interface的无缓存channel来串联起output和input，而每个环节都会new一个goroutine来执行对应操作，从而实现并行处理加速。
2. semaphore模式通过控制并发上限来避免资源耗光，通过设置一个指定信号量大小的有缓存channel来控制每次并发处理最大数（也就是同时存在的goroutine的数量）。信号量的方式是去有缓存的信号量channel里面获取信号，能获取到的话就new一个gorooutine来执行某个任务。
3. worker pool模式和信号量模式一样，也是通过控制并发上限来避免资源耗光。线程池的方式是先起好固定上限的goroutine，然后所有的goroutine从任务queue里面不断的取任务来执行。

可以看到信号量和线程池的差别是：1. 前者是每次都new新的goroutine，而后者是用的一个固定的goroutine池子。 

相同点就是：1. 都能够通过控制并发上限来避免资源耗光。2. 都使用了带缓存的channel。
## Tip
这周在熟悉代码的过程中接触到了context.WithCancel的使用方法。函数签名与源码如下：
```
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	c := newCancelCtx(parent)
	propagateCancel(parent, &c)
	return &c, func() { c.cancel(true, Canceled) }
}
```
return值为ctx和取消回调函数cancel，只要执行回调函数cancel，这时候调用context的Done()方法就能取到结果。可以通过这个方法实现父子协程的取消联动。
## Share
分享"Spark权威指南"第24章节“高级分析和机器学习概览”里对机器学习的几个常用场景总结：
1. 监督学习：根据数据项的各种特征预测每个数据项的标签。其中又分为以下2种任务类型：
  - 分类：训练一个算法来预测因变量的类别（属于离散的，有限的一组值）。最常见的就是辨别垃圾邮件：使用一组已经确定为垃圾邮件或者非垃圾邮件的历史电子邮件训练一个模型来分析这些邮件内容（单词和各种属性），并最终用这个模型来预测判定输入的邮件是否属于垃圾邮件，
  - 回归：训练一个算法来预测连续的变量（实数）。典型的例子：根据父母的身高预测他们孩子的身高。
2. 推荐系统：研究用户对于多种商品的显示偏好（通过评级）或者隐式偏好（通过观察到的行为），基于用户之间的相似性或商品之间的相似性来推荐给用户他们可能喜欢的商品。这里的商品是泛指，可能是一个电商场景的商品，也可能是一个推荐的视频。
3. 无监督学习：试图在一组给定的数据中寻找模式或者发现隐层结构，我个人理解就是让AI自行对数据进行聚类、分组。不同于监督学习，没有因变量（标签）来做预测。典型的例子：游戏公司基于在游戏中花费的时间/金钱等对用户进行聚类。产生的模型可能揭示休闲玩家与铁杆玩家完全不同的行为模式，并根据这些差异性给不同玩家提供不同建议或者奖励。
4. 图分析：研究给定顶点（对象）和边（表示这些对象之间的关系）的结构。例如google的原始网页推荐算法pageRank就是通过分析网站关系来对网页重要行进行排名的图算法。

一般来说机器学习的过程有如下步骤：
1. 搜集与任务有关的数据
2. 清洗数据
3. 执行特征工程使数据以适合的形式提供给算法。例如将数据转换为数值向量。
4. 使用数据的一部分作为训练集训练出一个或者多个模型，生成候选模型。
5. 使用从未被用作训练的数据子集来实际客观的评价结果，从而对步骤4生成的模型效果进行评估。
6. 使用步骤5中效果好的模型来落地生产，服务业务。

刚接触机器学习，看起来之前规划的“线上用户晚高峰qps预测”更符合监督学习里面的回归场景，后续再深入了解下。