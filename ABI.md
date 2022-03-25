# ABI 应用二进制接口

一个ABI定义了机器代码如何访问数据结构与运算程序，涵盖各种细节

- 数据类型的大小、布局和对齐;
- [调用约定](https://zh.wikipedia.org/wiki/%E8%B0%83%E7%94%A8%E7%BA%A6%E5%AE%9A)（控制着函数参数与返回值的传递方式、传递顺序），例如，是所有的参数都通过栈传递，还是部分参数通过寄存器传递；哪个寄存器用于哪个函数参数；通过栈传递的第一个函数参数是最先push到栈上还是最后；
- [系统调用](https://zh.wikipedia.org/wiki/%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8)的编码和一个应用如何向操作系统进行系统调用；
- 以及在一个完整的操作系统ABI中，[目标文件](https://zh.wikipedia.org/wiki/%E7%9B%AE%E6%A0%87%E6%96%87%E4%BB%B6)的二进制格式、程序库等等。

调用约定

- 极微参数或复杂参数独立部分的分配顺序
- 参数是如何被传递的（放置在堆栈上，或是寄存器中，亦或两者混合）
- 被调用者应保存调用者的哪个[寄存器](https://zh.wikipedia.org/wiki/%E5%AF%84%E5%AD%98%E5%99%A8)
- 调用[函数](https://zh.wikipedia.org/wiki/%E5%AD%90%E7%A8%8B%E5%BA%8F)时如何为任务准备[堆栈](https://zh.wikipedia.org/wiki/%E5%A0%86%E6%A0%88)，以及任务完成如何恢复

**[Go internal ABI specification](https://go.googlesource.com/go/+/refs/heads/dev.regabi/src/cmd/compile/internal-abi.md)**

**[Proposal: Create an undefined internal calling convention](https://go.googlesource.com/proposal/+/master/design/27539-internal-abi.md)**

**[Proposal: Register-based Go calling convention](https://go.googlesource.com/proposal/+/master/design/40724-register-calling.md)**

go1.17之前：函数的参数与返回值都通过栈来传递

优点：实现简单，不用担心底层cpu架构寄存器的差异，适合跨平台

缺点：牺牲了性能

![Untitled](ABI%20%E5%BA%94%E7%94%A8%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%8E%A5%20381cf/Untitled.png)

大多数平台上的大多数语言实现都使用基于寄存器的调用约定

go1.17之后：

go编译器会将前9个参数、前9个返回值通过寄存器传递，多出的参数、返回值则通过栈进行传递

参数的顺序对应使用寄存器的顺序为：AX、BX、CX、DI、SI、R8、R9、R10、R11

```go
package main

import (
	"fmt"
	"reflect"
	"runtime"
)

type config struct {
	foo []string
}

type ConfigOption func(*config)

func WithFoo(foo []string) ConfigOption {
	return func(cfg *config) {
		cfg.foo = foo
	}
}

var initFunc = runtime.FuncForPC(reflect.ValueOf(WithFoo(nil)).Pointer()).Name()

func main() {
	//go1.16  main.WithFoo.func1
	//go1.17  main.init.func1
	fmt.Println("Results:", initFunc)
}
```