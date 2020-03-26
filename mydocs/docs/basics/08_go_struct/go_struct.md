# Go的结构体

## 什么是结构体

Go 语言中数组可以存储同一类型的数据，但在结构体中我们可以为不同项 定义不同的数据类型。在Go中结构体类似于面向对象语言中的类。

结构体是由一系列具有相同类型或不同类型的数据构成的数据集合。

## 使用struct定义一个结构体

结构体定义需要使用 type 和 struct 语句。struct 语句定义一个新的数据类型，结构体中有一个或多个成员。type 语句设定了结构体的名称。结构体的格式如下：

```go
/*
type 结构体名字 struct {
	属性 类型
	属性 类型
}
 */
```

示例：

```go
package main

import "fmt"

// 大小写表示公有和私有
type Person struct {
	Name string
	Age  int
}

func main() {
	p := Person{"张三", 30}
	fmt.Println(p, p.Name, p.Age)
}
// {张三 30} 张三 30
```



## 创建匿名结构体

没有名字也没有type关键字，定义在函数内部。只是使用一次，把一堆属性放到一个变量中。

```go
package main

import "fmt"

func main() {
	a := struct {
		name string
		age  int
	}{name: "cwz", age: 19}
	fmt.Println(a, a.name, a.age)
}
```

## 结构体的零值

结构体是值类型，零值是属性的零值

```go
package main

import "fmt"

type Person struct {
	name string
	age  int
}

func main() {
	var p Person
	fmt.Println(p) // { 0}
	p.name = "cwz"  // 赋值
	p.age = 23
	fmt.Println(p, p.name, p.age) // {cwz 23} cwz 23
}
```

## 结构体的指针

```go
package main

import "fmt"

type Animal struct {
	name string
	age  int
}

func main() {
	a := &Animal{"gugu", 3}
	fmt.Println(*a, (*a).name, (*a).age)
}
// {gugu 3} gugu 3
```

## 匿名字段

在创建结构体时，字段可以只有类型，而没有字段名

```go
package main

import "fmt"

type Animal struct {
	string
	int
}

func main() {
	//a := Animal{"Aclo", 2}
	// 或者可以：
	var a1 Animal
	a1.string = "Aclo"
	a1.int = 2
	fmt.Println(a1)
}
```

匿名字段没有名字，但是匿名字段的名称默认是它的类型



## 嵌套结构体

结构体的字段也是一个结构体

```go
package main

import "fmt"

type Person struct {
	name string
	age  int
	hobby Hobby
}
type Hobby struct {
	id   int
	name string
}

func main() {
	p := Person{
		name:  "cwz",
		age:   24,
		hobby: Hobby{1, "阅读"},
	}
	fmt.Println(p)  // {cwz 24 {1 阅读}}
	p.hobby.name = "跑步"
	fmt.Println(p)  // {cwz 24 {1 跑步}}
}
```

## 提升字段

如果结构体中有匿名的结构体类型字段，该匿名结构体里的字段就称为提升字段。提升字段就像是属于外部结构体字段一样，可以用外部结构体直接访问。

```go
package main

import "fmt"

type Address struct {
	city, state string
}
type Person struct {
	name string
	age  int
	Address  // 匿名字段
}

func main() {
	var p Person
	p.name = "reese"
	p.age = 30
	p.Address = Address{
		city:  "Shanghai",
		state: "Jingan",
	}
	fmt.Println(p)
}

// {reese 30 {Shanghai Jingan}}
```

## 结构相等性

结构体是值类型

- 如果它的每一个字段都是可比较的，则该结构体也是可比较的。 如果两个结构体变量的对应字段相等，则这两个变量也是相等的
- 如果结构体包含不可比较的字段，则结构体变量也不可比较

