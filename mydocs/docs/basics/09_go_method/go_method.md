# Go的方法

## 什么是方法

方法是绑定给结构体的，和函数是有区别的。

属性 + 方法 ——> 类

语法：

```go
func (t Type) methodName(parameter list) {}
```

## 定义方法

```go
package main

import "fmt"

type Person struct {
	name string
	age  int
}

// 给结构体绑定方法
func (p Person) printName() {
	fmt.Println(p.name)
}

//普通函数
func printName(p Person)  {
	fmt.Println(p.name)
}

func main() {
	p := Person{
		name: "reese",
		age:  34,
	}
	p.printName()
}
```

Go不是一个纯粹的面向对象编程语言，而且Go也不支持类。因此，基于类型的方法是一种实现和类 相似行为的途径。

## 指针接收器和值接收器

之前函数的参数传递都是值进行传递，也就是会复制参数的值，我们想要修改传入的值，就需要传递一个指针。

```go
package main

import "fmt"

type Person struct {
	name string
	age  int
}

// 值接收器修改名字
func (p Person) changeName(name string) {
	p.name = name
	fmt.Println(p)  // {reese 19}
}

// 指针类型接收器修改年龄
func (p *Person) changeAge(age int) {
	(*p).age = age
	p.age = age
	fmt.Println(p)  // &{cwz 29}
}
func main() {
	// 值接收器，不会修改原来的，指针接收器才会修改原来的
	p := Person{"cwz", 19}
	(&p).changeName("reese")
	(&p).changeAge(29)
	fmt.Println(p)  // {cwz 29}
}
```

- 不管是值类型接收器还是指针类型接收器，都可以用值和指针来调用
- 指针接收器也可以被使用在如下场景：当拷贝一个结构体的代价过于昂贵时



## 匿名字段的方法

属于结构体的匿名字段的方法可以被直接调用

```go
package main

import "fmt"

type Hobby struct {
	id        int
	hobbyName string
}

type Person struct {
	name  string
	age   int
	Hobby // 匿名字段
}

// Hobby的绑定方法
func (h Hobby) printName() {
	fmt.Println(h.hobbyName)
}

// Person的绑定方法
func (p Person) printName() {
	fmt.Println(p.name)
}

func main() {
	p := Person{}
	p.hobbyName = "阅读"
	p.printName()  //继承，子类没有printName，就会调用父类的printName方法
	p.Hobby.printName() //就要调用父类的printName(面向对象中的super)
}
```



## 序列化

如果想要通过网络进行传输，我们就需要使用序列化协议，将 go 的结构体按照一种方式编码成字节串。在 go 里是通过给 struct 添加 tag 来实现的

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

type Animal struct {
	Name string
	Age  int
}

func main() {
	animals := []Animal{
		Animal{"dog", 3},
		Animal{"cat", 2},
	}
	res, err := json.Marshal(animals)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(string(res))  // [{"Name":"dog","Age":3},{"Name":"cat","Age":2}]

}
```

