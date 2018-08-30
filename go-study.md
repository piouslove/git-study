# go-study

## runtime
* golang编译器产生二进制代码，但这些代码运行在golang的runtime中（这部分的代码可以在 runtime 包中找到）。
* runtime 类似 Java 和 .NET 语言所用到的虚拟机，它负责管理包括内存分配、垃圾回收、栈处理、goroutine、channel、切片（slice）、map 和反射（reflection）等等。
* runtime 主要由 C 语言编写（Go 1.5 开始自举），并且是每个 Go 包的最顶级包。
* Go 拥有简单却高效的标记-清除回收器，目前 gccgo 还没有回收器，同时适用 gc 和 gccgo 的新回收器正在研发中。
* 使用一门具有垃圾回收功能的编程语言不代表你可以避免内存分配所带来的问题，分配和回收内容都是消耗 CPU 资源的一种行为。
* Go 的可执行文件都比相对应的源代码文件要大很多，这恰恰说明了 Go 的 runtime 嵌入到了每一个可执行文件当中；在部署到数量巨大的集群时，较大的文件体积也是比较头疼的问题。
* 因为 Go 具有像动态语言那样快速编译的能力，自然而然地就有人会问 Go 语言能否在 REPL（read-eval-print loop）编程环境下实现；Sebastien Binet 已经使用这种环境实现了一个 Go 解释器，你可以在这个[页面](https://github.com/sbinet/igo)找到。

## 开发环境
* IDE编辑器推荐 [Sublime Text](http://liteide.org/cn/documents/) 或 [LiteIDE](http://liteide.org/cn/documents/)。
* 目前可用的调试器是 gdb，最新版均以内置在集成开发环境 LiteIDE 和 GoClipse 中，但是该调试器的调试方式并不灵活且操作难度较大；如果你不想使用调试器，你可以按照下面的一些有用的方法来达到基本调试的目的：
	* 在合适的位置使用打印语句输出相关变量的值（`print/println` 和 `fmt.Print/fmt.Println/fmt.Printf`）。
	* 在 fmt.Printf 中使用下面的说明符来打印有关变量的相关信息：
		* `%+v` 打印包括字段在内的实例的完整信息
		* `%#v` 打印包括字段和限定类型名称在内的实例的完整信息
		* `%T` 打印某个类型的完整说明
	* 使用 panic 语句来获取栈跟踪信息（直到 panic 时所有被调用函数的列表）。
	* 使用关键字 defer 来跟踪代码执行过程。
* [gofmt](https://golang.org/cmd/gofmt/)自动[格式化](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/03.5.md)代码。
* [godoc](https://godoc.org/golang.org/x/tools/cmd/godoc) 工具会从 Go 程序和包文件中提取顶级声明的首行注释以及每个对象的相关注释，并生成相关文档。
* 工具 [cgo](http://golang.org/cmd/cgo) 提供了对 FFI（外部函数接口）的支持，能够使用 Go 代码安全地调用 C 语言库。

## GO程序的基本结构和要素
### 包
* 因式分解关键字导入，最好按照字母顺序排列包名，这样做更加清晰易读。
	```go
import (
   "fmt"
   "os"
)
	```
* 如果包名不是以 `.` 或 `/` 开头，如 `"fmt"` 或者 `"container/list"`，则 Go 会在全局文件进行查找；如果包名以 `./` 开头，则 Go 会在相对目录中查找；如果包名以 `/` 开头（在 Windows 下也可以这样使用），则会在系统的绝对路径中查找。
* 当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 private ）。
（大写字母可以使用任何 Unicode 编码的字符，比如希腊文，不仅仅是 ASCII 码中的大写字母）。因此，在导入一个外部包后，能够且只能够访问该包中导出的对象。

### 函数
* 符合规范的函数一般写成如下的形式：

```go
func functionName(parameter_list) (return_value_list) {
   …
}
```
	其中：
		* `parameter_list` 的形式为 `(param1 type1, param2 type2, …)`
		* `return_value_list` 的形式为 `(ret1 type1, ret2 type2, …)`
* 只有当某个函数需要被外部包调用的时候才使用大写字母开头，并遵循 [Pascal 命名法](https://baike.baidu.com/item/%E5%B8%95%E6%96%AF%E5%8D%A1%E5%91%BD%E5%90%8D%E6%B3%95/9464494?fr=aladdin)；否则就遵循[骆驼命名法](https://blog.csdn.net/f_zyj/article/details/51510085)，即第一个单词的首字母小写，其余单词的首字母大写。
* main 函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数（如果有 init() 函数则会先执行该函数）;main 函数既没有参数，也没有返回类型（与 C 家族中的其它语言恰好相反）。

### 注释
* 多行注释也叫块注释，均已以 /* 开头，并以 */ 结尾，且不可以嵌套使用，多行注释一般用于包的文档描述或注释成块的代码片段。
* 每一个包应该有相关注释，在 `package` 语句之前的块注释将被默认认为是这个包的文档说明，其中应该提供一些相关信息并对整体功能做简要的介绍。
* 一个包可以分散在多个文件中，但是只需要在其中一个进行注释说明即可；当开发人员需要了解包的一些情况时，自然会用 `godoc` 来显示包的文档说明。
* 在首行的简要注释之后可以用成段的注释来进行更详细的说明，而不必拥挤在一起；另外，在多段注释之间应以空行分隔加以区分。
```go
// Package superman implements methods for saving the world.
//
// Experience has shown that a small number of procedures can prove
// helpful when attempting to save the world.
package superman
```
* 几乎所有全局作用域的类型、常量、变量、函数和被导出的对象都应该有一个合理的注释；如果这种注释（称为文档注释）出现在函数前面，例如函数 Abcd，则要以 "Abcd..." 作为开头。
```go
// enterOrbit causes Superman to fly into low Earth orbit, a position
// that presents several possibilities for planet salvation.
func enterOrbit() error {
   ...
}
```













