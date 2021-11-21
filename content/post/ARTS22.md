---
title: "ARTS22"
date: 2021-11-21T21:38:14+08:00
---

## Algorithm
#### [剑指 Offer 17\. 打印从1到最大的n位数](https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof/)
```
func printNumbers(n int) []int {
	count := math.Pow(10, float64(n))
	result := make([]int, int(count-1))
	index := 1
	for index < int(count) {
		result[index-1] = index
		index++
	}
	return result
}
```
## Review
[5 Lessons That Golang Teaches To All Programmers](https://levelup.gitconnected.com/5-lessons-that-golang-teaches-to-all-programmers-71b332504cf2)
文章讲述了Go语言值得借鉴的几个特点：
1. 不过度设计。Go语言的设计理念确实一直是能简单就简单，less is more。不止是体现在文章里面举例的只有25个关键字、不提供while/for条件、不提供三元表达式等语言特性，还体现在官方一直宣扬的理念。比如不提供map和reduce等函数式常用方法、不鼓励使用运行时反射实现依赖注入而是通过wire生成代码的方式实现依赖注入。简单不过度设计就意味着代码可读性、性能好。
1. 尽量不动核心代码。这个大家工作都应该深有感触，核心代码尽量少动。
1. 使用Error处理错误，而不是Exception抛出。好处是让程序员充分考虑清楚整个数据流、错误场景。坏处就是代码里面一堆的 if err != nil。。
1. 不用OOP也能解决问题。不要太纠结于面向对象，重要的是代码的解耦、可复用性、可读性。
1. 多考虑考虑解决问题的各种方法。
## TIP
这周学会了怎么使用grafana来可视化查看ES的数据
1. 建立data sources，指定数据索引
2. 写query，查询出对应数据
3. 选择对应数据作为X/Y轴即可
## Share
[GO 编程模式：FUNCTIONAL OPTIONS](https://coolshell.cn/articles/21146.html)
[Golang中常见的option设计探讨](https://mp.weixin.qq.com/s/mzI8-KoRBhH-fGdfcyqI-w)
```
type Option func(*Server) error
func Protocol(p string) Option {
    return func(s *Server) error{
        s.Protocol = p
        // 可以进行其他校验，如果出错就return对应err
        return nil
    }
}
func Timeout(timeout time.Duration) Option {
    return func(s *Server) error{
        s.Timeout = timeout
    }
}
func MaxConns(maxconns int) Option {
    return func(s *Server) error{
        s.MaxConns = maxconns
    }
}
func TLS(tls *tls.Config) Option {
    return func(s *Server) error {
        s.TLS = tls
    }
}
func NewServer(addr string, port int, options ...func(*Server)) (*Server, error) {
  srv := Server{
    Addr:     addr,
    Port:     port,
    Protocol: "tcp",
    Timeout:  30 * time.Second,
    MaxConns: 1000,
    TLS:      nil,
  }
  for _, option := range options {
        if err := option(&srv); err != nil {
              return nil, err
    }
  }
  //...这里可以做一下非法参数校验，防止注入不在预期范围内的配置项
  return &srv, nil
}
// 使用方法
s3, _ := NewServer("0.0.0.0", 8080, Timeout(300*time.Second), MaxConns(1000))
```
