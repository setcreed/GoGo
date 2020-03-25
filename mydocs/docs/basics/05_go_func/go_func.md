# Go的函数

## 如何定义一个函数

go定义一个函数很简单，语法如下：

```go
func 函数名(参数1 类型，参数2 类型) 返回值 {
    函数体
}
```

简单写一个函数示例：

```go
package main

import "fmt"

func test(){
	fmt.Println("我是test函数")
}

func main() {
	test()
}

// 我是test函数
```

## 带参数的函数

```go
package main

import "fmt"

func test(a int, b int) {

	fmt.Println(a + b)
}

func main() {
	test(18, 19)
}
```

带参数的函数，两个参数类型相同，可以省略

```go
package main

import "fmt"

func test(a,  b int) {

	fmt.Println(a + b)
}

func main() {
	test(18, 19)
}
```

注意：go中只有位置参数，没有默认参数和关键字参数



##  有返回值

```go
package main

import "fmt"

func sum1(a, b int) int {    //需要指定返回值类型是什么
	return a + b

}

func main() {
	res := sum1(10, 20)
	fmt.Println(res)
}
```

我们可以给返回值命名，这个时候需要通过赋值的方式来更新结果，而且 return 可以不用带返回值

```go
func sum2(a int, b int) (res int) {
	res = a + b
	return
}
```

可以返回多个值

```go
package main

import "fmt"

func test(a string, b int) (string, int) {
	return a, b
}

func main() {
	fmt.Println(test("cwz", 3))
}
// cwz 3
```

## 可变参数

可变长参数，用来接收任意长度的参数

```go
package main

import "fmt"

func test(a ...int) {
	fmt.Println(a)
}

func main() {
	test(1, 3, 4)
}
// [1 3 4]
```

## 匿名函数

go 中我们也可以使用匿名函数，经常用在一些临时的小函数中：

```go
package main

import "fmt"

func test()  {
	// 匿名函数直接加括号
	func () {
		fmt.Println("我是匿名函数")
	}()
}

func main() {
	test()
}
// 我是匿名函数
```

## 函数类型

go 里边函数其实也是**一等公民**，函数本身也是一种类型，所以我们可以定义一个函数然后赋值给一个变量:

```go
package main

import "fmt"

func main() {
	res := func(a string) {fmt.Println(a)}
	res("hello golang")
	fmt.Printf("%T", res)
}

// hello golang
// func(string)
```



函数这个类型,它的参数，返回值，都是类型的一部分

```go
var a func()
var b func(a,b int)
var c func(a,b int)int
var d func(a,b int)(int,string)
// 这几个的函数类型是不一样的
```



给函数类型重命名，使用type

```go
package main

import "fmt"

type name string
func test(a name)  {
	fmt.Println(a)
}

func main() {
	test("cwz")
}
```





## 闭包函数

闭包函数满足两个条件：

- 定义在函数内部
- 函数体代码对外部作用域有引用

```go
package main

import "fmt"

func test() {
	// 只是一个内层函数
	a := func() {
		fmt.Println("我是内层函数")
	}
	a()
}

func main() {
	test()
}
// 我是内层函数
```

如下才是闭包函数：

```go
package main

import "fmt"

func test()  {
	su:="golang"
	addSu := func(name string) string {
		return name + su   // 这里使用到了 su 这个变量，所以 addSu 就是一个闭包
	}
	fmt.Println(addSu("hello, "))
}

func main() {
	test()
}

// hello, golang
```

go中没有装饰器语法糖，所以要利用闭包实现装饰器

```go
package main

import "fmt"
// 测试函数
func help(name string) {
	fmt.Println("原函数", name)
}

// 装饰器函数
func decorator(a func(name string)) func(string) {
	res:= func(name string) {
		fmt.Println("装饰器函数")   // 装饰器代码
		a(name)
	}
	return res
}

func main() {
	help := decorator(help)
	help("hello golang")
}
```

