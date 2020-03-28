# Go如何处理错误

## 错误处理

在python中使用`try / except`来进行异常的捕获和处理。还可以通过异常栈来追踪异常的调用信息从而帮助我们修复异常代码。

在go中，错误用内建的`error`类型来表示，错误值可以存储在变量里，作为函数的返回值。

示例：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Open("1.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(f.Name(), "成功打开")
}
```

在 go 的惯例中，一般函数多个返回值的最后一个值用来返回错误，返回 nil 表示没有错误，调用者通过检查返回的错误是否是 nil 就知道是否需要处理错误了。

## go的错误error类型

error 是 go 的一个内置的接口类型，比如你可以使用开发工具跳进去看下 error 的定义：

```go

// The error built-in interface type is the conventional interface for
// representing an error condition, with the nil value representing no error.
type error interface {
    Error() string
}
```

error 的定义很简单，只要我们自己实现了一个类型的 Error() 方法返回一个字符串，就可以当做错误类型了。举一个简单小例子， 比如计算两个整数相除，我们知道除数是不能为 0 的，这个时候我们就可以写个函数：

```go
package main

import (
	"errors" // 使用内置的 errors
	"fmt"
)


func Divide(a, b int) (int, error) {
	if b == 0 {
		return 0, errors.New("divide by zero")
	}
	return a / b, nil
}

func main() {
	// fmt.Println(testDefer())
	a, b := 1, 0
	res, err := Divide(a, b)
	if err != nil {
		fmt.Println(err) // error 类型实现了 Error() 方法可以打印出来
	}
	fmt.Println(res)
}
```





## defer语句

go 中提供了一个 defer 语句用来延迟一个函数(匿名函数)或者方法的执行，它会在函数执行完成(return)之后调用。一般为了防止代码里有资源泄露， 对于打开的资源比如文件等我们需要显式关闭，这种场合就是 defer 发挥作用最好的场景，也是 go 代码中使用 defer 最常用的场景。

```go
package main

import "fmt"

func main() {
	defer fmt.Println("我最后执行")      // 延迟执行
	fmt.Println("来了")
}

/*
输出结果：
来了
我最后执行
*/
```



函数里可以使用多个 defer 语句，如果有多个 defer 它们会按照后进先出(Last In First Out)的顺序执行。

```go
package main

import "fmt"

func testDefer() string {
	defer fmt.Println("defer 1")
	defer fmt.Println("defer 2")
	fmt.Println("函数体")
	return "test"
}

func main() {
	fmt.Println(testDefer())
}

/*
输出结果：
函数体
defer 2
defer 1
test

*/
```



## go的异常处理

go 的异常处理机制 panic(恐慌)/recover(恢复)，一般我们使用的是错误处理(error)而不是 panic。因为只有非常严重的场景 下才会发生 panic 导致代码退出

平常我们使用的 web 框架，一般即使出错了，我们也希望整个进程继续执行，而不是直接退出无法处理用户请求。 比如 python 的 web 框架，如果遇到了业务代码没有捕获的异常，框架会给我们捕获然后返回给客户端 500 的状态码表示代码有错。

go 里区分对待异常(panic)和错误(error)的，绝大部分场景下我们使用的都是错误，只有少数场景下发生了严重错误我们想让整个进程都退出了才会使用异常。

比如刚才除法函数的例子，如果我们碰到了个除数为 0 被认为是严重错误，也可以使用 panic 抛出异常：

```go
func Divide1(a, b int) int {
    if b == 0 {
        panic("divide by zero")
    }
    return a / b
}
```



如果我们传入除数0，但是又不想让进程退出呢？go 还提供了一个 recover 函数用来从异常中恢复，比如使用 recover 可以把一个 panic 包装成为 error 再返回，而不是让进程退出：

```go
func Divide2(a, b int) (res int, e error) {
    defer func() {
        if err := recover(); err != nil {
            e = fmt.Errorf("%v", err)
        }
    }()
    res = MustDivide(a, b)
    return // 命名返回值不用加上返回的参数
}
```

这样一来我们就捕获了 panic 异常并且返回了一个错误，代码也可以正常执行而不会退出。



## go定义自己的异常

在python中，可以通过继承Exception类来定义自己的业务异常。

在go中我们只需要自己定义一个结构体，然后实现`Error()`方法就实现了go的error接口。

```go
package errors

import (
	"fmt"
)

type ArticleError struct {
	Code    int64
	Message string
}

func (e *ArticleError) Error() string {
	return fmt.Sprintf("[ArticleError] Code=%d, Message=%s", e.Code, e.Message))
}

func NewArticleError(code int64, message string) error {
	return &ArticleError{
		Code:    code,
		Message: message,
	}
}
```

这里的web服务叫做Article服务，那么我们可以定义叫做ArticleError的错误类型。

如果出现了业务错误，你就可以调用 NewArticleError 函数并且传入你业务里定义的错误码和错误信息创建一个错误类型了。

总结一下：

- 对于一般不太严重的场景，返回错误值 error 类型
- 对于严重的错误需要整个进程退出的场景，使用 panic 来抛异常，及早发现错误
- 如果希望捕获 panic 异常，可以使用 recover 函数捕获，并且包装成一个错误返回

