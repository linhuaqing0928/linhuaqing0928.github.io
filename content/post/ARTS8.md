---
title: "ARTS8"
date: 2021-07-25T22:23:16+08:00
---

## Algorithm
#### [剑指 Offer 07\. 重建二叉树](https://leetcode-cn.com/problems/zhong-jian-er-cha-shu-lcof/)
这道题没有做出来，看了官方解答之后用Golang写一遍
递归法：
```
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func buildTree(preorder []int, inorder []int) *TreeNode {
	// 遍历中序数组并将数据放到map中，方便后续读取数据
	index := 0
	treeMap := make(map[int]int)
	for ; index < len(inorder); index++ {
		treeMap[inorder[index]] = index
	}
	head := buildTreeWithIndex(0, len(preorder)-1, 0, len(preorder)-1, preorder, inorder, treeMap)
	return head
}

func buildTreeWithIndex(preorderLeft, preorderRight, inorderLeft, inorderRight int, preorder []int, inorder []int, treeMap map[int]int) *TreeNode {
	if preorderLeft > preorderRight {
		return nil
	}

	preorderRoot := preorderLeft
	inorderRoot := treeMap[preorder[preorderRoot]]
	inorderLeftSize := inorderRoot - inorderLeft
	head := &TreeNode{preorder[preorderRoot], nil, nil}
	head.Left = buildTreeWithIndex(preorderLeft+1, preorderLeft+inorderLeftSize, inorderLeft, inorderRoot-1, preorder, inorder, treeMap)
	head.Right = buildTreeWithIndex(preorderLeft+inorderLeftSize+1, preorderRight, inorderRoot+1, inorderRight, preorder, inorder, treeMap)
	return head
}
```

## Review
[Zuul API Gateway](https://medium.com/geekculture/zuul-api-gateway-2bcdf4dd33e6)文章介绍了Netflix的Zuul网关在微服务架构中的位置以及作用

1. 微服务中的各个Microservice都是通过注册到服务治理框架上，调用方通过服务发现来拿到具体服务的IP/端口等信息才能调用到具体服务。
2. zuul网关职责：
  - 位于服务治理server之前，负责接收外部请求，然后根据path去服务治理服务上进行服务发现，最后将请求路由到具体的微服务app上。
  - 统一负责安全相关工作：对请求进行授权、认证等安全鉴权相关操作。
3. 因为zuul所处的位置并且有filter的功能，我们还可以在zuul上落地一些其他事情：
  - 在zuul上埋点进行数据采集，从而用于后续的分析。
  - 可以通过zuul切流对特定集群进行压力测试。
  - 对特定的静态资源请求直接在zuul上处理掉，减轻后端服务压力
## Tip

最近对自己的项目进行单元测试编写，接触了Golang标准库的mock功能。
1.  go get -u github.com/golang/mock/gomock安装gomock
1. go get -u github.com/golang/mock/mockgen安装mockgen
1. 使用mockgen对需要mock的interface生成mock文件：mockgen -source=xx.go -destination=xx_mock.go -package=xxx
1. 在单元测试方法中使用gomock引用刚刚生成的mock文件里面的方法和结构体，从而达到mock的目的。
生成的mock文件
```
import (
	reflect "reflect"

	gomock "github.com/golang/mock/gomock"
)

// MockTask is a mock of Task interface.
type MockTask struct {
	ctrl     *gomock.Controller
	recorder *MockTaskMockRecorder
}

// MockTaskMockRecorder is the mock recorder for MockTask.
type MockTaskMockRecorder struct {
	mock *MockTask
}

// NewMockTask creates a new mock instance.
func NewMockTask(ctrl *gomock.Controller) *MockTask {
	mock := &MockTask{ctrl: ctrl}
	mock.recorder = &MockTaskMockRecorder{mock}
	return mock
}

// EXPECT returns an object that allows the caller to indicate expected use.
func (m *MockTask) EXPECT() *MockTaskMockRecorder {
	return m.recorder
}

// GetTraceTaskID mocks base method.
func (m *MockTask) GetTraceTaskID(taskID int) (int, error) {
	m.ctrl.T.Helper()
	ret := m.ctrl.Call(m, "GetTraceTaskID", taskID)
	ret0, _ := ret[0].(int)
	ret1, _ := ret[1].(error)
	return ret0, ret1
}

// GetTraceTaskID indicates an expected call of GetTraceTaskID.
func (mr *MockTaskMockRecorder) GetTraceTaskID(taskID interface{}) *gomock.Call {
	mr.mock.ctrl.T.Helper()
	return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "GetTraceTaskID", reflect.TypeOf((*MockTask)(nil).GetTraceTaskID), taskID)
}
```
单元测试方法：
```
func TestGetDefaultTaskTraceId(t *testing.T) {
	// task interface mock
	ctrl := gomock.NewController(t)
	defer ctrl.Finish() 
	taskMock := rhino_service.NewMockTask(ctrl)
    taskMock.EXPECT().GetTraceTaskID(gomock.Eq(index)).Return(index, nil) // 进行mock打桩
}
```
## Share
最近在对自己之前的项目代码进行重构，这里记录一下最近重构的几个比较基础的点：
1. 注意数据安全：
  - 之前写代码图方便，struct里面的字段全是大写public的。这样子需要实例化的时候，直接一股脑把数据传进去就完事了。但是存在一个隐患就是，因为字段全是public的话，后续任何代码都可以直接访问并修改实例的内容，这样子很容易导致莫名其妙的bug。
  - 合理的方式应该是，struct内的字段默认全是小写private的。然后开一个NewXX的方法（类似于java里面的构造函数），在NewXX的方法入参里面对字段进行初始化。同时在外部确实需要查询或者修改字段的时候，再新增get/set方法。
2.  依赖注入(DI)，如果方法A依赖另一个功能B，应该通过注入的方式引入这个方法。这样子方便后续的扩容，同时也方便单元测试的落地。举个例子：在方法A里面需要调用结构体B的方法b1。
  - 比较不好的方法是在方法A里面直接实例化B然后调用b1方法。
  - 合理的方式应该是在方法A的形参里面传入B，然后在方法内通过形参B调用b1方法。
3. 面向接口编程。接着上面这个例子，更进一步的代码优化：
  - 在方法A的形参传入一个interface C。
  - C里面定义方法b1，然后B实现方法b1。
  - 这样子后续如果有新增需求：改变b1的操作内容。只需要新增一个结构体D，然后实现新的b1，最后在调用方法A的地方，将D的实例传入方法A即可，而不需要改变方法A。