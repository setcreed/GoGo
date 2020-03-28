# Go接口

## 接口

接口定义一个对象的行为，一系列方法的集合。

go通过接口实现了类型安全的鸭子类型（python之类的动态语言有鸭子类型）

我们定义一个Sleeper的接口

```go
// Sleeper 接口声明
type Sleeper interface {
    Sleep() // 声明一个 Sleep() 方法
}
```

然后定义两个结构体，一个Dog，一个Cat，都实现Sleep方法

```go
type Dog struct {
	Name string
}

func (d Dog) Sleep() {
	fmt.Printf("Dog %s is sleeping", d.Name)
}
type Cat struct {
	Name string
}

func (c Cat) Sleep() {
	fmt.Printf("Cat %s is sleeping", c.Name)
}
```

为了符合鸭子类型，函数的参数是一个接口类型而不是具体的 struct 类型。

```go
func AnimalSleep(s Sleeper)  {
	s.Sleep()
}
```

完整代码

```go
package main

import "fmt"

type Sleeper interface {
	Sleep()
}

type Dog struct {
	Name string
}

func (d Dog) Sleep() {
	fmt.Printf("Dog %s is sleeping\n", d.Name)
}
type Cat struct {
	Name string
}

func (c Cat) Sleep() {
	fmt.Printf("Cat %s is sleeping\n", c.Name)
}

func AnimalSleep(s Sleeper)  {
	s.Sleep()
}

func main() {
	var s Sleeper
	dog := Dog{Name:"大黄"}
	cat := Cat{Name:"饺子"}
	s = dog
	AnimalSleep(s)  // 使用 dog 的 Sleep()
	s = cat
	AnimalSleep(s)  // 使用 cat 的 Sleep()

	sleepList := []Sleeper{Dog{Name: "大黄"}, Cat{Name: "饺子"}}
	for _, s := range sleepList {
		s.Sleep()
	}
}
/*
Dog 大黄 is sleeping
Cat 饺子 is sleeping
Dog 大黄 is sleeping
Cat 饺子 is sleeping
*/
```

我们先声明了一个接口类型的值，只要实现了这个接口的 struct 变量，都可以赋值给它， 而调用方法的时候，go 会根据实际类型选择使用哪个 struct 的方法。



## 接口的内部表示

我们可以把接口看成内部的一个元祖（type，value）。type是接口底层的具体类型，而value是具体类型的值。

```go
package main

import (
	"fmt"
)

type Test interface {
	Tester()
}

type MyFloat float64

func (m MyFloat) Tester() {
	fmt.Println(m)
}

func describe(t Test) {
	fmt.Printf("接口类型 %T value %v\n", t, t)
}

func main() {
	var t Test
	f := MyFloat(23.4)
	t = f
	describe(t)
	t.Tester()
}

/*
输出结果：
接口类型 main.MyFloat value 23.4
23.4
*/
```

Test 接口只有一个方法Tester()，而MyFloat 类型实现了该接口。main函数中，

把变量f（MyFloat类型）赋值给了t（Test类型）。现在t的具体类型为MyFloat，而t的值为23.4。describe函数打印出了接口的具体类型和值。

## 空接口

没有包含方法的接口称为空接口。控接口表示为 `interface{}`。由于空接口没有方法，所以 所有类型都实现了空接口。

```go
//package main
//
//import (
//	"fmt"
//)
//
//type Test interface {
//	Tester()
//}
//
//type MyFloat float64
//
//func (m MyFloat) Tester() {
//	fmt.Println(m)
//}
//
//func describe(t Test) {
//	fmt.Printf("接口类型 %T value %v\n", t, t)
//}
//
//func main() {
//	var t Test
//	f := MyFloat(23.4)
//	t = f
//	describe(t)
//	t.Tester()
//}

package main

import "fmt"

func describe(i interface{}) {
	fmt.Printf("类型：%T, 值：%v\n", i, i)
}

func main() {
	s := "Hello World"
	describe(s)
	a := 12
	describe(a)
	st := struct {
		name string
	}{"reese"}
	describe(st)
}
/*
输出结果：
类型：string, 值：Hello World
类型：int, 值：12
类型：struct { name string }, 值：{reese}
*/
```

`describe(i interface{})`函数接收空接口作为参数，因此可以给这个函数传递任何类型。

## 接口嵌套

go的struct可以通过嵌套，实现代码复用

我们试一下，在文章开始，代码中申明了的Sleeper接口，再来声明一个接口Eater。有了睡和吃接口，在搞一个Lazy接口。

```go
package main

import "fmt"

// 申明Sleeper接口
type Sleeper interface {
	Sleep() // Sleep 方法
}

// 申明Eater接口
type Eater interface {
	Eat(footName string) // 申明Eat方法
}

// 申明Lazy接口
type Lazy interface {
	Sleeper
	Eater
}

type Dog struct {
	Name string
}

func (d Dog) Sleep() {
	fmt.Printf("Dog %s is sleeping\n", d.Name)
}
func (d Dog) Eat(footName string) {
	fmt.Printf("Dog %s is eating %s\n", d.Name, footName)
}

type Cat struct {
	Name string
}

func (c Cat) Sleep() {
	fmt.Printf("Cat %s is sleeping\n", c.Name)
}
func (c Cat) Eat(footName string) {
	fmt.Printf("Cat %s is eating %s\n", c.Name, footName)
}
func AnimalSleep(s Sleeper) {
	s.Sleep()
}

func main() {
	sleepList := []Lazy{Dog{"大黄"}, Cat{"饺子"}}
	footName := "food"
	for _, s := range sleepList {
		s.Sleep()
		s.Eat(footName)
	}
}
/*
输出结果：
Dog 大黄 is sleeping
Dog 大黄 is eating food
Cat 饺子 is sleeping
Cat 饺子 is eating food
*/
```

这样就实现了接口的嵌套，总结：

- go 可以声明接口，它包含了一系列方法声明
- struct 可以实现接口，只要一个 struct 实现了一个接口的所有方法，我们就说 struct 实现了这个接口（隐式的）
- 接口也可以通过嵌入来声明一个新的接口，比如 ReadWriter 内嵌了 Reader 和 Writer。
- go 提倡“小而美”的接口，然后通过嵌入来组合新接口



## 类型断言（type asset）

在使用接口的地方，我们可以传入一个具体的实现了接口的struct类型，但是怎么获取传入的到底是哪种struct类型呢？go中提供了类型断言来获取接口的底层值。

类型断言的语法格式如下：

```go
instance, ok := interfaceVal.(RealType) 
// 如果 ok 为 true 的话，接口值就转成了我们需要的类型
```

在上述的代码中加上类型断言

```go
package main

import "fmt"

// 申明Sleeper接口
type Sleeper interface {
	Sleep() // Sleep 方法
}

// 申明Eater接口
type Eater interface {
	Eat(footName string) // 申明Eat方法
}

// 申明Lazy接口
type Lazy interface {
	Sleeper
	Eater
}

type Dog struct {
	Name string
}

func (d Dog) Sleep() {
	fmt.Printf("Dog %s is sleeping\n", d.Name)
}
func (d Dog) Eat(footName string) {
	fmt.Printf("Dog %s is eating %s\n", d.Name, footName)
}

type Cat struct {
	Name string
}

func (c Cat) Sleep() {
	fmt.Printf("Cat %s is sleeping\n", c.Name)
}
func (c Cat) Eat(footName string) {
	fmt.Printf("Cat %s is eating %s\n", c.Name, footName)
}
func AnimalSleep(s Sleeper) {
	s.Sleep()
}

func main() {
	sleepList := []Lazy{Dog{"大黄"}, Cat{"饺子"}}
	footName := "food"
	for _, s := range sleepList {
		s.Sleep()
		s.Eat(footName)

		// 类型断言
		if dog, ok := s.(Dog); ok {
			fmt.Printf("I am a dog，name is %s\n", dog.Name)
		}
		if cat, ok := s.(Cat); ok {
			fmt.Printf("I am a cat，name is %s\n", cat.Name)
		}
	}
}
/*输出结果：
Dog 大黄 is sleeping
Dog 大黄 is eating food
I am a dog，name is 大黄
Cat 饺子 is sleeping
Cat 饺子 is eating food
I am a cat，name is 饺子
*/
```

在for 循环里边我们使用类型断言获取了接口值的真正类型。

