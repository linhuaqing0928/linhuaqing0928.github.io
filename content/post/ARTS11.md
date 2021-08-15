---
title: "ARTS11"
date: 2021-08-16T01:36:24+08:00
---

## Algorithm
本周听同组同学新人串讲，提到了计算表达式相关知识点，所以上leetcode找出题目，复习写了下。
#### [224\. 基本计算器](https://leetcode-cn.com/problems/basic-calculator/)
```
func calculate(s string) int {
	s = strings.ReplaceAll(s, " ", "")
	var result int
	intStack := list.New()
	intStack.PushBack(0)
	oprateStack := list.New()
	start := 0
	end := start + 1
	for end <= len(s) {
		if s[start:end] == "(" {
			oprateStack.PushBack("(")
			start = end
			end = start + 1
			continue
		}
		if s[start:end] == "-" || s[start:end] == "+" {
			if start > 0 && (s[start-1:start] == "-" || s[start-1:start] == "+" || s[start-1:start] == "(") {
				intStack.PushBack(0)
			}
			for oprateStack.Len() != 0 && oprateStack.Back().Value != "(" {
				topInt := intStack.Back()
				intLast := intStack.Remove(topInt).(int)
				topInt1 := intStack.Back()
				intLast1 := intStack.Remove(topInt1).(int)
				topOprate := oprateStack.Back()
				oprateLast := oprateStack.Remove(topOprate).(string)
				tempResult := caculateInnner(intLast1, intLast, oprateLast)
				intStack.PushBack(tempResult)
			}
			oprateStack.PushBack(s[start:end])
			start = end
			end = start + 1
			continue
		}
		if s[start:end] == ")" {
			for oprateStack.Len() != 0 && oprateStack.Back().Value != "(" {
				topInt := intStack.Back()
				intLast := intStack.Remove(topInt).(int)
				topInt1 := intStack.Back()
				intLast1 := intStack.Remove(topInt1).(int)
				topOprate := oprateStack.Back()
				oprateLast := oprateStack.Remove(topOprate).(string)
				tempResult := caculateInnner(intLast1, intLast, oprateLast)
				intStack.PushBack(tempResult)
			}
			if oprateStack.Len() != 0 && oprateStack.Back().Value == "(" {
				topOprate := oprateStack.Back()
				oprateStack.Remove(topOprate)
			}
			start = end
			end = start + 1
			continue
		}
		if end+1 <= len(s) && s[end:end+1] != "+" && s[end:end+1] != "-" && s[end:end+1] != "(" && s[end:end+1] != ")" {
			end += 1
			continue
		}
		tempInt, _ := strconv.Atoi(s[start:end])
		intStack.PushBack(tempInt)
		start = end
		end = start + 1
	}
	for oprateStack.Len() != 0 {
		topInt := intStack.Back()
		intLast := intStack.Remove(topInt).(int)
		topInt1 := intStack.Back()
		intLast1 := intStack.Remove(topInt1).(int)
		topOprate := oprateStack.Back()
		oprateLast := oprateStack.Remove(topOprate).(string)
		tempResult := caculateInnner(intLast1, intLast, oprateLast)
		intStack.PushBack(tempResult)
	}
	topInt := intStack.Back()
	result = intStack.Remove(topInt).(int)
	return result
}

func caculateInnner(a, b int, sumbol string) int {
	if sumbol == "+" {
		return a + b
	} else {
		return a - b
	}
}

func TestCalCulate(t *testing.T) {
	var input string
	var result int
	var expected int

	input = "1 + 1"
	result = calculate(input)
	expected = 2
	fmt.Println(result)
	if result != expected {
		t.Errorf("TestcalCulate failed! expected %d, got %d", expected, result)
	}

	input = " 2-1 + 2 "
	result = calculate(input)
	expected = 3
	fmt.Println(result)
	if result != expected {
		t.Errorf("TestcalCulate failed! expected %d, got %d", expected, result)
	}

	input = "(1+(4+5+2)-3)+(6+8)"
	result = calculate(input)
	expected = 23
	fmt.Println(result)
	if result != expected {
		t.Errorf("TestcalCulate failed! expected %d, got %d", expected, result)
	}

	input = "2147483647"
	result = calculate(input)
	expected = 2147483647
	fmt.Println(result)
	if result != expected {
		t.Errorf("TestcalCulate failed! expected %d, got %d", expected, result)
	}

	input = "-2+ 1"
	result = calculate(input)
	expected = -1
	fmt.Println(result)
	if result != expected {
		t.Errorf("TestcalCulate failed! expected %d, got %d", expected, result)
	}

	input = "-(3+(4+5))"
	result = calculate(input)
	expected = -12
	fmt.Println(result)
	if result != expected {
		t.Errorf("TestcalCulate failed! expected %d, got %d", expected, result)
	}

	input = "(7)-(0)+(4)"
	result = calculate(input)
	expected = 11
	fmt.Println(result)
	if result != expected {
		t.Errorf("TestcalCulate failed! expected %d, got %d", expected, result)
	}
}
```
## Review
[Golang and clean architecture](https://itnext.io/golang-and-clean-architecture-19ae9aae5683)
文章讲述了他认为比较好的Golang项目结构：
  - entity放具体的数据结构，就是model层。
  - userCase放具体的业务逻辑，就是service。
  - handler进行request解析、校验、调用service、并返回response，就是controller层。
  - 文章还有一层Frameworks and Adapters，就是entity层和数据库的关联具体实现层。这个其实就是传统的DAO层。

文章整体看下来，没有太大的收获，就是MVC那一套。业务代码中其实关键的还是层与层之前的扩容性、边界性。
```
.
├── api
│   ├── handler
│   │   ├── handler.go
│   │   ├── helpers.go
│   │   └── model.go
│   └── middleware
│       └── middleware.go
├── cloudformation
│   └── apigateway
│       ├── apig.yaml
│       └── create_apigateway.sh
├── entity
│   ├── entity.go
│   ├── errors.go
│   └── validations.go
├── go.mod
├── go.sum
├── lambdas
│   ├── authorizer
│   │   ├── deploy.sh
│   │   ├── main.go
│   │   └── template.yaml
│   └── users
│       ├── deploy.sh
│       ├── env.json
│       ├── main.go
│       └── template.yaml
├── Makefile
├── openapi.yaml
├── pkg
│   └── jwtparser
│       ├── jwtparser.go
│       └── jwtparser_test.go
├── README.md
├── repository
│   ├── dynamodb
│   │   └── dynmodb.go
│   ├── mockdatabase
│   │   └── mockdatabase.go
│   └── mysql
│       ├── init.sql
│       ├── model.go
│       └── mysql.go
└── usecase
    └── users
        ├── repository.go
        ├── service.go
        ├── usecase.go
        ├── users.go
        └── users_test.go
17 directories, 33 files
```
## Tip
本周工作过程需要实现将dataFrame一行数据扩展成多行的功能。例如原来数据是如下：
```
+--------------------+--------------------+
|    logid           |                products|
+--------------------+--------------------+
|20210815173559010|111111 222222 333333 444444|
|20210815173559011|222222 333333 444444 555555|
```
需要转换为：
```
+--------------------+--------------------+
|    logid           |                product|
+--------------------+--------------------+
|20210815173559010|111111|
|20210815173559010|222222|
|20210815173559010|333333|
|20210815173559010|444444|
|20210815173559011|222222|
|20210815173559011|333333|
|20210815173559011|444444|
|20210815173559011|555555|
```
解决方案是
1. 转换为RDD
2. 使用map函数，将每一个row转换为ArrayBuffer[Row]
3. 再使用flatMap(_.toIterator)展开为RDD

伪代码如下：
```
def XXX(logTime: String): Unit = {
    val RDD1 = RDD2
      .map(row => {
        expand(row)
      })
      .flatMap(_.toIterator)
  }
private def expand(row: Row): ArrayBuffer[Row] = {
    val productRows = new ArrayBuffer[Row]()
    val logID = row.getString(0)
    val products = row.getString(1).split(" ")
    products.foreach(x => {
      productRows.append(Row(logID, x))
    })
    productRows
  }
```
参考资料：[Split one row into multiple rows of dataframe](https://stackoverflow.com/questions/56523026/split-one-row-into-multiple-rows-of-dataframe)
## Share
##### 对烂代码要尽量保持零容忍
之前工作中自己维护的一个项目，前段时间抽时间对代码进行了一些优化，部分模块代码的扩展性和可测性都有了一定的提升。
后面来了一些临时小需求，工期比较赶的情况下没有好好的进行设计，偷懒加了一些垃圾代码。结果这周五做国际化需求的时候，发现因为之前的那些垃圾代码，扩展性变得很差，要支持国际化需求就要加更多的垃圾代码。
这种情况相信大家都会遇到，新服务一开始代码结构的扩展性、可测性都相对较好，结果后面因为“工期赶”、“小需求”、“先这样实现，后空再优化”等等借口，慢慢的“屎山”就形成了。
从每一行代码、每一个需求做起，尽量保持“代码洁癖”。遇到烂代码，不要“下一次”。