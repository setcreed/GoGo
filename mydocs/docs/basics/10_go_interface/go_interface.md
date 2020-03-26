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