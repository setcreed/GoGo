# Go过程式编程

## Go的控制流程语句

和很多编程语言类似，Go也提供了if/else等 控制流程语句。

## if 语句

if/else 使用起来和 python 的非常像，go的语法如下：

```go
{
    if 条件 {
		//符合上面条件的执行
	}
	else if 条件{
		//符合上面条件的执行
	}else {
		// 不符合上面的条件执行这个
}
```

举例说明：

```go
package main

import "fmt"

func main() {
	var age int = 30
	if age < 18 {
		fmt.Println("未成年")
	}else {
		fmt.Println("成年")
	}
}

```

需要注意：

if 条件只能是一个 bool 值或者返回 bool 值的表达式，而不能是int

## 循环

与其他语言不同的是，go只提供了for关键字来实现循环，没有while。

语法如下：

```go
// 常规用法，和其他语言类似
for 初始化; 条件; 自增自减 {
  循环体的内容
}

// 无限循环，block 会被一直重复执行
for {
  block
}

// 实现while循环，block 一直执行直到 表达式为false
for 条件 {
  block
}
```

几个例子

```go
package main

import "fmt"

func main() {
	var i int = 0
	for ;i < 10 ; i++ {
		fmt.Println(i)
	}
}



package main

import "fmt"

func main() {
	i := 0
	for ; i < 10; {
		fmt.Println(i)
		i++
	}
}

```

九九乘法表：

```go
package main

import "fmt"

func main() {
	for i := 1; i < 10; i++ {
		for j := 1; j < i+1; j++ {
			fmt.Printf("%dx%d=%d\t", j, i, i*j)
		}
		fmt.Println()
	}
}
```

