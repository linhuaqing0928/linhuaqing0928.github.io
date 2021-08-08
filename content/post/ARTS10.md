---
title: "ARTS10"
date: 2021-08-08T22:49:27+08:00
---

## Algorithm
#### [剑指 Offer 27\. 二叉树的镜像](https://leetcode-cn.com/problems/er-cha-shu-de-jing-xiang-lcof/)
比较简单的二叉树题目，分别使用递归和栈实现了下
```
import "container/list"

//   Definition for a binary tree node.
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

// 递归算法
func mirrorTree(root *TreeNode) *TreeNode {
	if root == nil {
		return nil
	}
	if root.Left == nil && root.Right == nil {
		return root
	}
	temp := root.Left
	root.Left = root.Right
	root.Right = temp
	if root.Right != nil {
		mirrorTree(root.Right)
	}
	if root.Left != nil {
		mirrorTree(root.Left)
	}
	return root
}

// 使用栈解法
func mirrorTree1(root *TreeNode) *TreeNode {
	if root == nil {
		return nil
	}
	if root.Left == nil && root.Right == nil {
		return root
	}
	list := list.New()
	list.PushBack(root)
	for list.Len() != 0 {
		first := list.Back()
		firstEle := list.Remove(first).(*TreeNode)
		if firstEle.Left != nil || firstEle.Right != nil {
			temp := firstEle.Left
			firstEle.Left = firstEle.Right
			firstEle.Right = temp
		}
		if firstEle.Left != nil {
			list.PushBack(firstEle.Left)
		}
		if firstEle.Right != nil {
			list.PushBack(firstEle.Right)
		}
	}
	return root
}
```
## Review
[Why you need to avoid using append in Go](https://daryl-ng.medium.com/why-you-need-to-avoid-using-append-in-go-8c8502b5d0da)
文章对比了Golang中对slice进行add的几种操作的性能
1. 申明一个空切片，然后使用append进行添加元素操作。
1. 使用make初始化一个固定长度的切片，然后通过下标进行元素添加。
1. 使用make初始化一个固定长度的切片，然后使用append进行元素添加。

最终对比下来，方案2/3的性能是相近的，而方案1的性能特别差。说明主要时间是花在切片扩容上。Golang切片的具体扩容代码可以看[slice.go](https://github.com/golang/go/blob/master/src/runtime/slice.go)的growslice方法，因为Golang做了比较多的性能优化，所以相对比较复杂。
代码：
```
func WithAppend() []string {
   var l []string
   for i := 0; i < 100; i++ {
      l = append(l, "x")
   }

   return l
}
func WithAssignAlloc() []string {
   l := make([]string, 100)
   for i := 0; i < 100; i++ {
      l[i] = "x"
   }

   return l
}
func WithAppendAlloc() []string {
   l := make([]string, 0, 100)
   for i := 0; i < 100; i++ {
      l = append(l, "x")
   }
   return l
}
BenchmarkWithAppend
BenchmarkWithAppend-8           863949       1322 ns/op     4080 B/op        8 allocs/op
BenchmarkWithAppendAlloc
BenchmarkWithAppendAlloc-8     2543119        514 ns/op     1792 B/op        1 allocs/op
BenchmarkWithAssignAlloc
BenchmarkWithAssignAlloc-8     2343424        523 ns/op     1792 B/op        1 allocs/op
```
## Tip
这周学习了日志服务的基础架构，核心点：
1. 在每一台服务器上都有日志agent，业务代码产生日志后，会批量发送到日志服务器。
1. 全量日志内落hdfs
1. 错误日志落es，索引是一些固定的信息：IP、实例名、服务名、日志代码行数等。方便后续错误日志的快速查询。
## Share
之前写过一篇关于[httpMock](https://www.jianshu.com/p/545963b593de)的单元测试实践文章，里面提到HttpMock的原理是：Activate函数中通过http.DefaultTransport = DefaultTransport修改了所有通过http/net包发送的请求的transport。所以本周学习了一下Golang的net/http包的[client.go源码](https://github.com/golang/go/blob/master/src/net/http/client.go)，了解一下一个httpClient是如何运作的。
1. httpClient的结构体：
```
type Client struct {
	// RoundTripper是一个接口，对外只提供一个RoundTrip方法（发送request请求）
	Transport RoundTripper

	// 是否重定向的方法判断，默认defaultCheckRedirect是重定向超过10次就停止，并报error
	CheckRedirect func(req *Request, via []*Request) error

	// cookie
	Jar CookieJar

	// 超时时间设置
	Timeout time.Duration
}
```
2. 不管是post，还是get/head等方法，都是调用了httpClient的Do方法。例如Post：
```
func Post(url, contentType string, body io.Reader) (resp *Response, err error) {
	return DefaultClient.Post(url, contentType, body)
}
func (c *Client) Post(url, contentType string, body io.Reader) (resp *Response, err error) {
	req, err := NewRequest("POST", url, body)
	if err != nil {
		return nil, err
	}
	req.Header.Set("Content-Type", contentType)
	return c.Do(req)
}
```
3. 来看一下Do方法做了啥，Do代码比较多，这里就不贴了。
  - 把重定向的request都append到一个[]*Request切片中。
  - 通过调用c.send，最终调用到httpClient结构体中的Transport的RoundTrip方法，进行request发送。
  - 拿到RoundTrip的返回值：resp, didTimeout, err。如果报错或者超时，则终止此次Do请求。如果否，继续下一步
  - 调用redirectBehavior方法，判断是否是否需要进行重定向。如果需要的话，则再次进入for循环。
4. redirectBehavior做了啥：主要就是根据上一次request发送得到的response的StatusCode判断是否需要重定向。代码如下：
```
func redirectBehavior(reqMethod string, resp *Response, ireq *Request) (redirectMethod string, shouldRedirect, includeBody bool) {
	switch resp.StatusCode {
	case 301, 302, 303:
		redirectMethod = reqMethod
		shouldRedirect = true
		includeBody = false

		// RFC 2616 allowed automatic redirection only with GET and
		// HEAD requests. RFC 7231 lifts this restriction, but we still
		// restrict other methods to GET to maintain compatibility.
		// See Issue 18570.
		if reqMethod != "GET" && reqMethod != "HEAD" {
			redirectMethod = "GET"
		}
	case 307, 308:
		redirectMethod = reqMethod
		shouldRedirect = true
		includeBody = true

		// Treat 307 and 308 specially, since they're new in
		// Go 1.8, and they also require re-sending the request body.
		if resp.Header.Get("Location") == "" {
			// 308s have been observed in the wild being served
			// without Location headers. Since Go 1.7 and earlier
			// didn't follow these codes, just stop here instead
			// of returning an error.
			// See Issue 17773.
			shouldRedirect = false
			break
		}
		if ireq.GetBody == nil && ireq.outgoingLength() != 0 {
			// We had a request body, and 307/308 require
			// re-sending it, but GetBody is not defined. So just
			// return this response to the user instead of an
			// error, like we did in Go 1.7 and earlier.
			shouldRedirect = false
		}
	}
	return redirectMethod, shouldRedirect, includeBody
}
```
参考资料：
[Go - http.Client源码分析](https://juejin.cn/post/6844903922637733901)

[Golang http之client源码详解](https://blog.csdn.net/skh2015java/article/details/89403057)