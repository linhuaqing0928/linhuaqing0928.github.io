---
title: "ARTS28"
date: 2022-01-09T23:33:57+08:00
---

## Algorithm
#### [剑指 Offer 05\. 替换空格](https://leetcode-cn.com/problems/ti-huan-kong-ge-lcof/)
```
func replaceSpace(s string) string {
	builder := strings.Builder{}
	for _, rune1 := range s {
		if rune1 != ' ' {
			builder.WriteRune(rune1)
		} else {
			builder.WriteString("%20")
		}
	}
	return builder.String()
}
```
## Review
[Best practices to communicate between microservices](https://irfanyusanif.medium.com/how-to-communicate-between-microservices-7956ed68a99a)
文章标题党，主要讲了怎么用C#发布和消费基于RabbitMQ的异步事件消息，内容比较简单。

作者的一个观点是微服务之间应该尽量都用异步消息来通讯，这个我是不认同的。大部分RPC调用还是同步堵塞型的，比较可控而且client能够及时根据response结果进行后续的正常或者异常逻辑处理。
## TIP
这周工作过程中需要建立一个grafana大盘，数据来源是ES。先来简单看下ES数据的数据格式：
```
CpuRateAvg | QPS | Date | ServiceName
0.3 | 1000 | 2021-11-07T00:00:00.000Z | Service_A
0.4 | 2000 | 2021-11-08T00:00:00.000Z | Service_A
0.3 | 1000 | 2021-11-07T00:00:00.000Z | Service_B
0.4 | 2000 | 2021-11-08T00:00:00.000Z | Service_B
```
现在想要在grafana大盘上，画出以下2个数据展示大盘：
1. 画折线图 其中X轴为时间 Y轴为CPU数值 并且以ServiceName为颗粒度，画出多条CPU折线图。
1. 画折线图 其中X轴为时间 Y轴为QPS数值 并且以ServiceName为颗粒度，画出多条QPS折线图。

同时因为service比较多，还需要能够维护一个全局变量ServiceName，在切换ServiceName的时候2个折线图能同时切换为同一个服务所属的折线图。

经过摸索，配置操作如下：
1. 在dashboard的Variables里面新建一个变量，Type为Query（也就是从ES数据中查询出具体有哪些ServiceName）。query内容为：{"find":"terms","field":"ServiceName"}。Data Source配置为ES索引对应的数据源，refresh配置为On Time Range Change。 这样子这个全局变量就是ES数据里面真实存在的服务名。
2. 新建一个Panel。然后添加一个query1，Query里面的查询语句写上：ServiceName: \$ServiceName 这个query的意思就是根据全局变量ServiceName当前的实参，从ES中查询出ServiceName== \$ServiceName的数据。对应到关系型数据库的sql就是，select * from XXX where ServiceName = xxx
3. query1的metrics设置为Max、CpuRateAvg。然后添加2个group by设置，第一个为Terms、ServiceName，第二个为Date Histogram、Date。 这样子就能出现一个期望的CPU折线图了。
4. 重复步骤2-3创建一个QPS折线图。
## Share
继续学习“操作系统32讲”
### L4 操作系统接口
操作系统接口（system_call）概念定义：接口表现为函数调用，又由系统提供，所以称为系统调用。

POSIX：Portable Operating System Interface of Unix （IEEE制定的一个标准族）[POSIX](https://pubs.opengroup.org/onlinepubs/9699919799.2018edition/)
### L5 系统调用的实现
首先，应用层代码不应该能够随意访问内存的任意地址（包含操作系统所在的内存地址），不然会有内存安全问题（比如随意读取操作系统层面的缓存，拿到root密码等信息），但是应用层代码又要能够调用操作系统接口（system_call保存在操作系统的内容空间段）。这里引出2个问题：
1. 怎么能隔离开应用层代码和操作系统代码 保障内存安全？
1. 隔离开之后，又怎么能让应用层代码能够成功调用system_call呢？
#### 操作系统怎么隔离内存，实现内存安全
操作系统区分内核态和用户态：通过一种处理器“硬件设计”来区分。具体过程：
1. 操作系统启动过程，在head.s代码中会通过初始化GDT（段寄存器访问内存之前会先到GDT表里面去查表，拿到要执行的指令的具体内存地址）表来给所有内存标记上具体哪段内存是内核端（对应的DPL为0），哪段是用户段（对应的DPL为3）。
1. 操作系统在取指执行的时候是根据CS：IP来决定去哪个内存块取指令执行的，而处理器的硬件设计用CS的最低两位来标识当前CPL：0代表内核态，3代表用户态。
1. 处理器在执行CS：IP的时候会去检查CPL（Current Privilege Level CS的最低2位）和DPL（Descriptor Privilege Level 段寄存器去GDT表取实际指令内存的时候能拿到这个值） 只有DPL大于等于CPL，才能访问这块内存。操作系统在用户态和内核态之间切换的时候，实际上就是不断的切换当前CPL的值。而DPL是在一开始GDT初始化的时候就决定了是0（内核段）还是3（用户段）
1. 最终表现出来的就是：内核态可以访问任何数据，用户态不能访问内核数据。
#### 应用程序如何访问内核段地址
硬件层面提供了“主动进入内核的方法”：通过中断指令int（int 0x80）
1. int指令将使CS中的CPL改为0，“进入内核”
1. 这是用户程序发起调用内核代码的唯一方式。

所以系统调用的核心其实就是：
1. 用户程序中包含一段int指令的代码
1. 操作系统写中断处理，获取想调程序的编号
1. 操作系统根据编号执行相应代码。

在从printf(...)应用层代码来看，实际上就是
```
#include <unistd.h>
_syscall3(int, write, int, fd, const char *buf, off_t, count)
#define _syscall3(type, name,...) type
name(...) \
{ __asm__ ("int 0x80" : "=a"(__res)...}
```
这里的C代码，看不太懂 大概意思就是将一个系统调用号(__NR__write 用来区分是哪个操作系统系统接口来调用的)置给eax寄存器，然后调用int 0x80。调用int 0x80之后就能进入内核。
##### int 0x80中断的处理
int 0x80做了啥：去IDT表查询要处理的中断处理函数地址 然后跳过去
