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



## switch / case

与C语言中的switch类似，但是go中改善了许多，不需要在每一个case中加上break，go语言的switch语法如下：

```go
switch var1 {
    case val1:
        ...
    case val2:
        ...
    default:
        ...
}
```

变量 var1 可以是任何类型，而 val1 和 val2 则可以是同类型的任意值。类型不被局限于常量或整数，但必须是相同的类型；或者最终结果为相同类型的表达式。

多个条件使用 `&&`（与）和 `||`（或）

注意：

- default语句 是可选的，表示 如果上面的条件都不符合，就给一个默认行为
- fallthrough  表示 无条件执行下一个case中的代码（用的很少）

小例子1：

```go
package main

import "fmt"

func main() {
	var day int = 3
	switch day {
	case 0,6:
		fmt.Println("周末")
	case 1,2,3,4,5:
		fmt.Println("工作日")
	default:
		fmt.Println("不合法")
	}
}
// 工作日
```

例子2：

```go
package main

import "fmt"

func main() {
	var a, b int = 10, 20
	a, b = b, a
	switch  {
	case a < b:      // case后面可以是表达式
		fmt.Println("a < b")
	case a > b:
		fmt.Println("a > b")

	}
}
```

