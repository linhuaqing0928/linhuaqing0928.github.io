---
title: "Golang单元测试实战：httpMock"
date: 2021-07-16T17:48:07+08:00
---

## 单元测试定义
之前看过一篇文章，里面讲到单元测试的2个原则：

- Unit tests should be pretty lightweight and run fast since they don’t depend on anything external to the unit of code being tested
- The only way they can fail is by changing the unit of code they are testing

记录一下最近对自己的代码进行单元测试实践
## 过程
### 被测代码
```
package http_request_demo

import (
	"errors"
	"io/ioutil"
	"net/http"
)

func getAPIResponse(url string) (string, error) {
	var err error
	request, err := http.NewRequest("GET", url, nil)
	if err != nil {
		return "", err
	}
	myClient := &http.Client{}
	response, err := myClient.Do(request)
	if err != nil {
		return "", err
	}
	defer response.Body.Close()
	if response.StatusCode != 200 {
		return "", errors.New("response not 200!")
	}
	bodyBytes, err := ioutil.ReadAll(response.Body)
	if err != nil {
		return "", err
	}
	return string(bodyBytes), nil
}
```
这个简单的函数主要做了以下几件事：
1. 构造http request
2. 用http client发起请求
3. 对请求动作的error进行判空、response的状态码判断、然后从body中read数据

### 如何单元测试
#### 真实发起请求并测试
拿到这个函数最开始就觉得很简单，直接拿万能的百度上手，梭哈完事
```
func TestGetAPIResponse(t *testing.T) {
	var api string
	var responseBody string
	var err error

	// first test
	api = "http://www.baidu.com"
	responseBody, err = getAPIResponse(api)
	if err != nil {
		t.Errorf("first test failed! err: %v", err)
	}
	if responseBody == "" {
		t.Errorf("first test failed! Unexpect response!")
	}
}
```
go test跑一下，完美
```
=== RUN   TestGetAPIResponse
--- PASS: TestGetAPIResponse (0.10s)
PASS
coverage: 73.3% of statements
ok      github.com/linhuaqing0928/golang-demo/http_mock_demo    0.319s
```
#### 问题
但是这时候覆盖率之后73.3% 再往上提升就麻烦了，而且明显是不符合文章开头说的2个单元测试的原则
1. 如果www.baidu.com哪天挂了，我们这个UT就会fail 因为他还依赖了www.baidu.com这个第三方 而且这个第三方不是我们可以控制的。
2. 要提升覆盖率的话，我们需要构造http.NewRequest("GET", url, nil)返回error、response.StatusCode != 200等场景，而因为我们无法控制www.baidu.com的返回，所以覆盖率基本就提升不上去了。

怎么解决这个问题呢，很容易想到几个mock方案：
1. 找一台服务器，上面搭建一个自己的服务 然后自己定制化的根据接收到的request内容，返回自定义的内容。  -- 这个方案确实一定程度解决了这个问题，但是存在2个问题：
  - 成本太高，需要维护一个新的服务
  - 跑UT的机器必须和这台服务器网络是通的。
2. 从根本上解决问题，不再发起网络请求。在client发起request请求的时候，直接拦截然后进行response返回的模拟。 -- 本文的主角httpmock出现了 

#### httpmock
直接贴github地址：https://github.com/jarcoal/httpmock  
readme讲解的很清楚了，使用方法基本就是以下几步：
1. 调用Activate方法启动httpmock环境
2. 通过httpmock.RegisterResponder方法进行mock规则注册。
3. 这时候再通过http client发起的请求就都会被httpmock拦截，如果匹配到刚刚注册的规则就会按照注册的内容返回对应response。  --- 这里感觉httpmock有一点不太好用，那就是如果请求没有命中规则就会错误返回，而不是走真实请求。需要把整个过程涉及到的所有请求都注册上去。
4. 在defer里面调用DeactivateAndReset结束mock 
```
func TestFetchArticles(t *testing.T) {
  httpmock.Activate()
  defer httpmock.DeactivateAndReset()

  // Exact URL match
  httpmock.RegisterResponder("GET", "https://api.mybiz.com/articles",
    httpmock.NewStringResponder(200, `[{"id": 1, "name": "My Great Article"}]`))

  // Regexp match (could use httpmock.RegisterRegexpResponder instead)
  httpmock.RegisterResponder("GET", `=~^https://api\.mybiz\.com/articles/id/\d+\z`,
    httpmock.NewStringResponder(200, `{"id": 1, "name": "My Great Article"}`))

  // do stuff that makes a request to articles
  ...

  // get count info
  httpmock.GetTotalCallCount()

  // get the amount of calls for the registered responder
  info := httpmock.GetCallCountInfo()
  info["GET https://api.mybiz.com/articles"] // number of GET calls made to https://api.mybiz.com/articles
  info["GET https://api.mybiz.com/articles/id/12"] // number of GET calls made to https://api.mybiz.com/articles/id/12
  info[`GET =~^https://api\.mybiz\.com/articles/id/\d+\z`] // number of GET calls made to https://api.mybiz.com/articles/id/<any-number>
}
```
#### 实战
在自己的单元测试里面使用httpmock
```
func TestGetAPIResponse(t *testing.T) {
	var api string
	var responseBody string
	var err error

	// url nil test
	api = "://新"
	responseBody, err = getAPIResponse(api)
	if err == nil {
		t.Errorf("url nil test failed! err: %v", err)
	}
	if responseBody != "" {
		t.Errorf("url nil test failed! Unexpect response!")
	}

	// http mock test
	var mockResponse string
	api = "http://www.baidu.com"
	mockResponse = "mock response body"
	httpmock.Activate()
	defer httpmock.DeactivateAndReset()
	httpmock.RegisterResponder("GET", api, httpmock.NewStringResponder(200, string(mockResponse)))
	responseBody, err = getAPIResponse(api)
	if err != nil {
		t.Errorf("second test failed! err: %v", err)
	}
	if responseBody != mockResponse {
		t.Errorf("second test failed! Unexpect response!")
	}

	// request fail test
	api = "http://www.baidu.com/fail"
	httpmock.RegisterResponder("GET", api,
		func(r *http.Request) (*http.Response, error) {
			return &http.Response{
				Status:        strconv.Itoa(200),
				StatusCode:    200,
				ContentLength: -1,
			}, errors.New("fail error!")
		})
	responseBody, err = getAPIResponse(api)
	if err == nil {
		t.Errorf("request fail test failed! err: %v", err)
	}
	if responseBody != "" {
		t.Errorf("request fail test failed! Unexpect response! response: %s", responseBody)
	}

	// wrong status test
	api = "http://www.baidu.com/wrongstatus"
	mockResponseWrong := "mock response body"
	httpmock.RegisterResponder("GET", api, httpmock.NewStringResponder(404, string(mockResponseWrong)))
	responseBody, err = getAPIResponse(api)
	if err == nil {
		t.Errorf("wrong request test failed! err: %v", err)
	}
	if responseBody != "" {
		t.Errorf("wrong request test failed! Unexpect response! response: %s", responseBody)
	}
}
// === RUN   TestGetAPIResponse
// 2021/07/16 16:14:53 RoundTripper returned a response & error; ignoring response
// --- PASS: TestGetAPIResponse (0.00s)
// PASS
// coverage: 93.3% of statements
// ok      github.com/linhuaqing0928/golang-demo/http_mock_demo    0.458s
```
可以看到因为可以定制化返回结果，所以我们的覆盖率能够达到93.3% 同时最重要的是，我们达到了开头说的单元测试的2个原则。

### httpmock原理解析
1. Activate函数中通过http.DefaultTransport = DefaultTransport修改了所有通过http/net包发送的请求的transport
2. DefaultTransport通过调用NewMockTransport方法实例化了一个MockTransport来代替DefaultTransport。这个MockTransport实现了http包中的RoundTripper接口。[transport源码](DefaultTransport)
3. 再来看一下MockTransport的结构体和RegisterResponder注册函数：
```
type MockTransport struct {
	mu               sync.RWMutex
	responders       map[internal.RouteKey]Responder
	regexpResponders []regexpResponder
	noResponder      Responder
	callCountInfo    map[internal.RouteKey]int
	totalCallCount   int
}

func (m *MockTransport) RegisterResponder(method, url string, responder Responder) {
	if isRegexpURL(url) {
		m.registerRegexpResponder(regexpResponder{
			origRx:    url,
			method:    method,
			rx:        regexp.MustCompile(url[2:]),
			responder: responder,
		})
		return
	}

	key := internal.RouteKey{
		Method: method,
		URL:    url,
	}

	m.mu.Lock()
	m.responders[key] = responder
	m.callCountInfo[key] = 0
	m.mu.Unlock()
}
```
可以看到其实就是在MockTransport中维护一组map
  - key是正则匹配的路由
  - value是则是type Responder func(*http.Request) (*http.Response, error)

4. 然后RegisterResponder的时候就是往这个map里面塞内容

## 最后
以上就是对http的使用和源码简单解读，大家可以看到就算引入了httpmock，最后的代码覆盖率还是达不到100%  因为bodyBytes, err := ioutil.ReadAll(response.Body)这个部分我们构造不了err的场景。

读一下http的response源码：
```
    // Body represents the response body.
	//
	// The response body is streamed on demand as the Body field
	// is read. If the network connection fails or the server
	// terminates the response, Body.Read calls return an error.
	//
	// The http Client and Transport guarantee that Body is always
	// non-nil, even on responses without a body or responses with
	// a zero-length body. It is the caller's responsibility to
	// close Body. The default HTTP client's Transport may not
	// reuse HTTP/1.x "keep-alive" TCP connections if the Body is
	// not read to completion and closed.
	//
	// The Body is automatically dechunked if the server replied
	// with a "chunked" Transfer-Encoding.
	//
	// As of Go 1.12, the Body will also implement io.Writer
	// on a successful "101 Switching Protocols" response,
	// as used by WebSockets and HTTP/2's "h2c" mode.
	Body io.ReadCloser
```
就会发现这个报错的出现场景：If the network connection fails or the server terminates the response, Body.Read calls return an error。 这个就不在我们单元测试能覆盖的范围内了，所以这个覆盖率无法满足就是合理的。