---
title: "Golang单元测试实战二：依赖注入与面向接口改造"
date: 2021-08-01T22:01:24+08:00
---

## 单元测试
- Unit tests should be pretty lightweight and run fast since they don’t depend on anything external to the unit of code being tested
- The only way they can fail is by changing the unit of code they are testing
## 过程
待单元测试代码：
```
func (f FileSDK) NewFile() string {
	myClient := ThridPClient{}
	_, err := myClient.GetRemoteFile()
	// do something
	if err.Error() == "error1" {
		return "default1.txt"
	}
	if err.Error() == "error2" {
		return "default2.txt"
	}
	return "default3.txt"
}

```
其中ThridPClient是一个第三方客户端SDK。很简单的，我们会想到构造不同client的场景得到不同err，然后覆盖这个方法的不同场景路径。
但是存在以下问题：
 - 假如第三方SDK对我们不可见，我们无法知道什么场景会返回什么类型的err
 - 如果第三方SDK的GetRemoteFile方法内容变了，那我们这个单元测试很可能会失败。

为了达到文章开头说的单元测试的2个原则，我们需要对以上的业务代码进行改造，提高他的可测性。
## 代码改造
工作过程中，我们经常会说这个代码没有可测性。很明显上述的代码是没有可测性的，原因在于2点
- 第三方client的依赖不是注入的，而是在function内部直接实例化的
- 由于Golang的结构体没有子类、父类多态的概念，所以没有办法通过一个mock子类继承client然后重写GetRemoteFile方法的方式实现mock。需要对代码进行面向接口的改造。

基于以上2点，我们对代码进行以下改造：
```
type FileSDK struct {
	fileClient FileClient
}

func NewFileSDK(fileClient FileClient) *FileSDK {
	return &FileSDK{
		fileClient: fileClient,
	}
}

type FileClient interface {
	GetRemoteFile() (string, error)
}

func (f FileSDK) NewFile() string {
	_, err := f.fileClient.GetRemoteFile()
	// do something
	if err.Error() == "error1" {
		return "default1.txt"
	}
	if err.Error() == "error2" {
		return "default2.txt"
	}
	return "default3.txt"
}
```
可以看到我们做了以下2点改造：
1. 声明一个FileClient的interface，里面包含GetRemoteFile方法。
2. 通过依赖注入的方式，将interface注入到方法中。

通过以上改造之后，再进行单元测试就简单多了。我们看下单元测试的代码。
```
type ThridPClientMock struct {
	fileType int
}

func (c ThridPClientMock) GetRemoteFile() (string, error) {
	if c.fileType == 1 {
		return "", errors.New("error1")
	}
	if c.fileType == 2 {
		return "", errors.New("error2")
	}
	return "xxx.txt", errors.New("")
}

func TestNewFile(t *testing.T) {
	var fileClient ThridPClientMock
	var fileSDK FileSDK
	var result string

	fileClient = ThridPClientMock{1}
	fileSDK = *NewFileSDK(fileClient)
	result = fileSDK.NewFile()
	if result != "default1.txt" {
		t.Errorf("TestNewFile failed: %v", result)
	}

	fileClient = ThridPClientMock{2}
	fileSDK = *NewFileSDK(fileClient)
	result = fileSDK.NewFile()
	if result != "default2.txt" {
		t.Errorf("TestNewFile failed: %v", result)
	}

	fileClient = ThridPClientMock{3}
	fileSDK = *NewFileSDK(fileClient)
	result = fileSDK.NewFile()
	if result != "default3.txt" {
		t.Errorf("TestNewFile failed: %v", result)
	}

}
```
可以看到，我们可以通过自己实现一个ThridPClientMock，然后通过NewFileSDK将这个mock类的实例注入到我们fileSDK中。因为mock类是我们自己实现的，可以自由控制GetRemoteFile的路由规则，从而覆盖不同err返回值的单元测试场景，并且完全不受第三方SDK的代码实现影响。也就是实现了文章开头说的单元测试的2个原则。
## Golang Mock
以上代码改造之后，我们的mock类是自己手写实现的。大家会发现还是挺麻烦的，所以Golang很暖心的实现了mockgen和gomock，可以帮我们快速生成interface的mock代码。简单使用方法如下：
1. go get -u github.com/golang/mock/gomock安装gomock
2. go get -u github.com/golang/mock/mockgen安装mockgen
3. 使用mockgen对需要mock的interface生成mock文件：mockgen -source=xx.go -destination=xx_mock.go -package=xxx
4. 在单元测试方法中使用gomock引用刚刚生成的mock文件里面的方法和结构体，从而达到mock的目的。

我们来看一下生成的mock代码：
```
import (
	reflect "reflect"

	gomock "github.com/golang/mock/gomock"
)

// MockFileClient is a mock of FileClient interface.
type MockFileClient struct {
	ctrl     *gomock.Controller
	recorder *MockFileClientMockRecorder
}

// MockFileClientMockRecorder is the mock recorder for MockFileClient.
type MockFileClientMockRecorder struct {
	mock *MockFileClient
}

// NewMockFileClient creates a new mock instance.
func NewMockFileClient(ctrl *gomock.Controller) *MockFileClient {
	mock := &MockFileClient{ctrl: ctrl}
	mock.recorder = &MockFileClientMockRecorder{mock}
	return mock
}

// EXPECT returns an object that allows the caller to indicate expected use.
func (m *MockFileClient) EXPECT() *MockFileClientMockRecorder {
	return m.recorder
}

// GetRemoteFile mocks base method.
func (m *MockFileClient) GetRemoteFile() (string, error) {
	m.ctrl.T.Helper()
	ret := m.ctrl.Call(m, "GetRemoteFile")
	ret0, _ := ret[0].(string)
	ret1, _ := ret[1].(error)
	return ret0, ret1
}

// GetRemoteFile indicates an expected call of GetRemoteFile.
func (mr *MockFileClientMockRecorder) GetRemoteFile() *gomock.Call {
	mr.mock.ctrl.T.Helper()
	return mr.mock.ctrl.RecordCallWithMethodType(mr.mock, "GetRemoteFile", reflect.TypeOf((*MockFileClient)(nil).GetRemoteFile))
}
```
使用方式如下：
```
func TestNewFile(t *testing.T) {
	var fileSDK FileSDK
	var result string
	var fileClient *MockFileClient

	ctrl := gomock.NewController(t)
	defer ctrl.Finish()

	fileClient = NewMockFileClient(ctrl)
	fileClient.EXPECT().GetRemoteFile().Return("", errors.New("error1")) // 进行mock打桩
	fileSDK = *NewFileSDK(fileClient)
	result = fileSDK.NewFile()
	if result != "default1.txt" {
		t.Errorf("TestNewFile failed: %v", result)
	}

	fileClient = NewMockFileClient(ctrl)
	fileClient.EXPECT().GetRemoteFile().Return("", errors.New("error2")) // 进行mock打桩
	fileSDK = *NewFileSDK(fileClient)
	result = fileSDK.NewFile()
	if result != "default2.txt" {
		t.Errorf("TestNewFile failed: %v", result)
	}

	fileClient = NewMockFileClient(ctrl)
	fileClient.EXPECT().GetRemoteFile().Return("", errors.New("error3")) // 进行mock打桩
	fileSDK = *NewFileSDK(fileClient)
	result = fileSDK.NewFile()
	if result != "default3.txt" {
		t.Errorf("TestNewFile failed: %v", result)
	}
}
```
可以看到gomock通过Controller实现了路由控制，而我们只需要使用EXPECT和Return方法就可以实现路由注册，真的很方便。推荐大家使用。