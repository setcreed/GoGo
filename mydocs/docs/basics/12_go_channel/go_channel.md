# Go的信道（channel）

## 什么是信道

信道是Go的协程之间通信的管道。信道就相当于一个变量。

## 定义一个信道

所有的信道都只关联了一个类型。信道只能运输这种类型的数据，不能运输其他类型的数据。

信道的零值是`nil`。

```go
package main

import "fmt"

func main() {
	var a chan int // 定义一个信道只能运输int类型的数据
	if a == nil {
		fmt.Println("信道的零值是nil")
		a = make(chan int) // 定义一个信道只能运输int类型的数据
		fmt.Printf("此时信道的类型是%T", a)
	}
}

/*
输出结果：
信道的零值是nil
此时信道的类型是chan int
*/
```



## 通过信道发送和接收数据

语法如下：

```go
data := <- a    // 读取信道 a
a <- data // 写入信道a，箭头指向a
```

信道的发送和接收默认是阻塞的。当数据发送到信道时，程序会在发送数据的语句处阻塞，直到有其他的Go协程从信道读取到数据，才会解除阻塞。当读取信道的数据时，如果没有其他的协程把数据写入到这个这个信道时，读取过程是阻塞的。

代码示例：

```go
package main

import "fmt"

func test(a chan int) {
	fmt.Println("我是test函数")
	a <- 1
}

func main() {
	a := make(chan int)
	go test(a)
	<-a
	fmt.Println("我是main函数")
}

/*
输出结果：
我是test函数
我是main函数
*/
```



## 死锁

使用信道是要考虑的一个重要因素是死锁（Deadlock）。如果一个协程发送数据给一个信道，而没有其他的协程从该信道中接收数据，那么程序在运行时会遇到死锁，并触发 panic 。

同样地，如果一个协程从一个信道中接收数据，而没有其他的协程向这个信道发送数据，那么程序同样造成死锁，触发 panic 。

```go
package main

func main() {
	ch := make(chan int)
	ch <- 2
}
```

此时会报错：

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
        /home/cwz/GOPATH/src/day05/s3.go:185 +0x50

```



## 单向信道

数据既可以发送到双向信道，也可以从双向信道中读取。同样也可以创建单向信道，即只能发送数据或只能接收数据的信道。

```go
package main

import "fmt"

func senData(sendCh chan<- int) {
	sendCh <- 10
}

func main() {
	sendCh := make(chan<- int)
	go senData(sendCh)
	fmt.Println(<-sendCh)     // 报错
}
```

我们创建了一个只写的信道sendch，`chan<- int`表示只能发送数据。在main函数中，我们试图从一个只写的信道中读取数据，结果报错了。

但是创建一个只写通道有什么用？又不能读取数据。

**这就是信道转型的用途。可以将双向信道转换为只写或只读信道，但是反过来却不行。**

```go
package main

import "fmt"

func senData(sendCh chan<- int) {
	sendCh <- 10
}

func main() {
	a := make(chan int)
	go senData(a)
	fmt.Println(<-a) // 输出10
}
```

在main函数中创建了一个双向通道a，`go senData(a)` a被作为参数传递给协程`sendData`。`senData(sendCh chan<- int)`函数通过形参`sendCh chan<- int`将该信道转换成只写信道。所以在`sendData`中该信道为只写信道，而在主协程中该信道为双向信道。

## 关闭信道

发送者可以关闭信道以及通知接受者将不会再发送数据给信道

在从信道接收数据时，接受者可以通过一个额外的变量来检测信道是否已经被关闭。

```go
v, ok := <-ch
```

上面的语句ok返回true表示成功的接收到了发送数据，如果ok返回false则表示信道已经被关闭。从已经关闭的信道中读取到的数据为信道类型的零值。

如从一个被关闭的int信道中读取数据，那么将得到0。

```go
package main

import "fmt"

func producer(chnl chan int) {
	for i := 0; i < 10; i++ {
		chnl <- i
	}
	close(chnl)
}

func main() {
	ch := make(chan int)
	go producer(ch)
	for {
		v, ok := <-ch
		if ok == false {
			break
		}
		fmt.Println("接收", v, ok)
	}
}
```

在上面的程序中，协程producer向信道chnl中写入0到9并关闭该信道。主协程在进行无线for循环，并在循环中检测ok的值判断是否已经被关闭。如果ok是false表示信道已经被关闭，则通过break退出循环。否则接收数据并打印ok的值。

## range 遍历信道

range for可以用来接收一个信道中的数据，知道该信道关闭

```go
package main

import "fmt"

func producer(chnl chan int) {
	for i := 0; i < 10; i++ {
		chnl <- i
	}
	close(chnl)
}

func main() {
	ch := make(chan int)
	go producer(ch)
	for v := range ch {
		fmt.Println("接收", v)
	}
}
```

`for range`不断从信道ch中接收数据，知道该信道关闭，一旦ch关闭，循环自动退出。



## 缓冲信道

之前的信道是基本的无缓冲区的信道，对一个无缓冲区的信道进行发送 / 写入接收 / 读取数据是实时阻塞的。事实上我们可以创建一个带缓冲区的通道，朝通道中发送数据时，只有当缓冲区满的情况下才会阻塞；类似的，只有从通道中读取数据，只有读到缓冲区空为止才会阻塞。

创建带缓冲区的信道

```go
ch := make(chan type, capacity)  // capacity指定缓冲区的大小
```

示例：

```go
package main

import "fmt"

func main() {
	ch := make(chan string, 2)
	ch <- "name"
	ch <- "reese"
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
/*
name
reese
*/
```

在这个例子中，我们创建了一个缓冲区大小为2的信道，这样我们就可以往信道中写入2个字符串后才被阻塞。

缓冲信道中的死锁

```go
package main

import "fmt"

func main() {
	ch := make(chan string, 2)
	ch <- "name"
	ch <- "reese"
	ch <- "cwz"
	fmt.Println(<-ch)
	fmt.Println(<-ch)
}
```

在上面的例子中，我们往一个缓冲区为2的信道中写入了3个元素，当我们三个元素写入时因为已经达到最大的容量所以协程会阻塞掉，直到有其他协程从channel中读取出来，但是我们这个例子中没有其他协程进行这个工作，所以就会造成死锁的出现，程序会打印下面信息：

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
        /home/cwz/GOPATH/src/day05/s3.go:252 +0x8d

```



缓冲信道的参数

capacity是指缓冲通道能容纳的最大元素的个数，这个值是我们创建缓冲通道时用make函数指定的。length是指当前缓冲通道中的元素个数：

```go
package main

import "fmt"

func main() {
	ch := make(chan string, 3)
	ch <- "name"
	ch <- "reese"
	fmt.Println("capacity is", cap(ch))
	fmt.Println("length is", len(ch))
	fmt.Println("read value", <-ch)
	fmt.Println("new length is", len(ch))
}
/*
输出结果：
capacity is 3
length is 2
read value name
new length is 1
*/
```

上面的例子中，创建了一个capacity为3的信道，可以容纳3个string元素。当写入了2个元素带通道中时，这时候的长度就是2，然后又从信道中读出1个元素，现在信道的长度就是1。