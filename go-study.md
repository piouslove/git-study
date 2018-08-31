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
* 符合规范的函数一般写成如下的形式,其中：
	* `parameter_list` 的形式为 `(param1 type1, param2 type2, …)`
	* `return_value_list` 的形式为 `(ret1 type1, ret2 type2, …)`

```go
func functionName(parameter_list) (return_value_list) {
   …
}
```
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

### 类型
* 使用 var 声明的变量的值会自动初始化为该类型的零值。
* 类型定义了某个变量的值的集合与可对其进行操作的集合。
* 类型可以是基本类型，如：int、float、bool、string；结构化的（复合的），如：struct、array、slice、map、channel；只描述类型的行为的，如：interface。
* 结构化的类型没有真正的值，它使用 nil 作为默认值。
使用 type 关键字可以定义你自己的类型，你可能想要定义一个结构体，但是也可以定义一个已经存在的类型的别名，如：`type IZ int`；这里并不是真正意义上的别名，因为使用这种方法定义之后的类型可以拥有更多的特性，且在类型转换时必须显式转换。
* 每个值都必须在经过编译后属于某个类型（编译器必须能够推断出所有值的类型），因为 Go 语言是一种静态类型语言。
* 由于 Go 语言不存在隐式类型转换，因此所有的转换都必须显式说明，就像调用一个函数一样（类型在这里的作用可以看作是一种函数）：`valueOfTypeB = typeB(valueOfTypeA)`

## Go 程序的一般结构
1. 在完成包的 import 之后，开始对常量、变量和类型的定义或声明。
2. 如果存在 init 函数的话，则对该函数进行定义（这是一个特殊的函数，每个含有该函数的包都会首先执行这个函数）。
3. 如果当前包是 main 包，则定义 main 函数。
4. 然后定义其余的函数，首先是类型的方法，接着是按照 main 函数中先后调用的顺序来定义相关函数，如果有很多函数，则可以按照字母顺序来进行排序。

## 常量
* 因为在编译期间自定义函数均属于未知，因此无法用于常量的赋值，但内置函数可以使用，如：len()。
* 反斜杠 `\` 可以在常量表达式中作为多行的连接符使用。
* 每遇到一次 `const` 关键字，`iota` 就重置为 0 。
* 常量之所以为常量就是恒定不变的量，因此我们无法在程序运行过程中修改它的值；如果你在代码中试图修改常量的值则会引发编译错误。 

## 变量
* 因式分解关键字的写法一般用于声明全局变量。
* 当一个变量被声明之后，系统自动赋予它该类型的零值：int 为 0，float 为 0.0，bool 为 false，string 为空字符串，指针为 nil；所有的内存在 Go 中都是经过初始化的。
* 一个变量（常量、类型或函数）在程序中都有一定的作用范围，称之为作用域；如果一个变量在函数体外声明，则被认为是全局变量，可以在整个包甚至外部包（被导出后）使用，不管你声明在哪个源文件里或在哪个源文件里调用该变量。
* 在函数体内声明的变量称之为局部变量，它们的作用域只在函数体内，参数和返回值变量也是局部变量；控制结构中声明的变量的作用域只在相应的代码块内；一般情况下，局部变量的作用域可以通过代码块（用大括号括起来的部分）判断。
* 尽管变量的标识符必须是唯一的，但你可以在某个代码块的内层代码块中使用相同名称的变量，则此时外部的同名变量将会暂时隐藏（结束内部代码块的执行后隐藏的外部同名变量又会出现，而内部同名变量则被释放），你任何的操作都只会影响内部代码块的局部变量。
* 所有像 int、float、bool 和 string 这些基本类型都属于值类型，使用这些类型的变量直接指向存在内存中的值；像数组（第 7 章）和结构（第 10 章）这些复合类型也是值类型。
* 一个引用类型的变量 r1 存储的是 r1 的值所在的内存地址（数字），或内存地址中第一个字所在的位置。
* 如果 r1 的值被改变了，那么这个值的所有引用都会指向被修改后的内容。
* 在 Go 语言中，指针属于引用类型，其它的引用类型还包括 slices ，maps 和 channel ；被引用的变量会存储在堆中，以便进行垃圾回收，且比栈拥有更大的内存空间。
* 使用操作符 := 可以高效地创建一个新的变量，称之为初始化声明；它只能被用在函数体内，而不可以用于全局变量的声明与赋值。
* `_`实际上是一个只写变量，你不能得到它的值；这样做是因为 Go 语言中你必须使用所有被声明的变量，但有时你并不需要使用从一个函数得到的所有返回值。
* Go 对于值之间的比较有非常严格的限制，只有两个类型相同的值才可以进行比较，如果值的类型是接口，它们也必须都实现了相同的接口。
* 对于布尔值的好的命名能够很好地提升代码的可读性，例如以 is 或者 Is 开头的 isSorted、isFinished、isVisible，使用这样的命名能够在阅读代码的获得阅读正常语句一样的良好体验。
* 由于 UTF-8 编码对占用字节长度的不定性，Go 中的字符串也可能根据需要占用 1 至 4 个字节；Go 这样做的好处是不仅减少了内存和硬盘空间占用，同时也不用像其它语言那样需要对使用 UTF-8 字符集的文本进行编码和解码。
* 字符串是一种值类型，且值不可变，即创建某个文本后你无法再次修改这个文本的内容；更深入地讲，字符串是字节的定长数组。
* Go 中使用 `strings` 包来完成对字符串的主要操作。
	* HasPrefix 判断字符串 s 是否以 prefix 开头，HasSuffix 判断字符串 s 是否以 suffix 结尾。
	* Contains 判断字符串 s 是否包含 substr `strings.Contains(s, substr string) bool`
	* Index 返回字符串 str 在字符串 s 中的索引（str 的第一个字符的索引），-1 表示字符串 s 不包含字符串 str。
	* LastIndex 返回字符串 str 在字符串 s 中最后出现位置的索引（str 的第一个字符的索引），-1 表示字符串 s 不包含字符串 str。
	* Replace 用于将字符串 str 中的前 n 个字符串 old 替换为字符串 new，并返回一个新的字符串，如果 n = -1 则替换所有字符串 old 为字符串 new。
	* Count 用于计算字符串 str 在字符串 s 中出现的非重叠次数。
	* Repeat 用于重复 count 次字符串 s 并返回一个新的字符串。
	* ToLower 将字符串中的 Unicode 字符全部转换为相应的小写字符；ToUpper 将字符串中的 Unicode 字符全部转换为相应的大写字符。
	* strings.Fields(s) 将会利用 1 个或多个空白符号来作为动态长度的分隔符将字符串分割成若干小块，并返回一个 slice，如果字符串只包含空白符号，则返回一个长度为 0 的 slice。
	* strings.Split(s, sep) 用于自定义分割符号来对指定字符串进行分割，同样返回 slice。
	* Join 用于将元素类型为 string 的 slice 使用分割符号来拼接组成一个字符串：`strings.Join(sl []string, sep string) string`
	* 函数 strings.NewReader(str) 用于生成一个 Reader 并读取字符串中的内容，然后返回指向该 Reader 的指针。
* 与字符串相关的类型转换都是通过 `strconv` 包实现的
	* 任何类型 T 转换为字符串总是成功的。
	* `strconv.Itoa(i int) string` 返回数字 i 所表示的字符串类型的十进制数。
	* `strconv.FormatFloat(f float64, fmt byte, prec int, bitSize int) string` 将 64 位浮点型的数字转换为字符串，其中 `fmt` 表示格式（其值可以是 'b'、'e'、'f' 或 'g'），prec 表示精度，`bitSize` 则使用 32 表示 `float32`，用 64 表示 `float64`。
	* 将字符串转换为其它类型 tp 并不总是可能的，可能会在运行时抛出错误 `parsing "…": invalid argument`。
	* `strconv.Atoi(s string) (i int, err error)` 将字符串转换为 int 型。
	* `strconv.ParseFloat(s string, bitSize int) (f float64, err error)` 将字符串转换为 float64 型。
* [time](http://docs.studygolang.com/pkg/time/) 包为我们提供了一个数据类型 `time.Time`（作为值使用）以及显示和测量时间和日期的功能函数。
```go
package main
import (
	"fmt"
	"time"
)

var week time.Duration
func main() {
	t := time.Now()
	fmt.Println(t) // e.g. Wed Dec 21 09:52:14 +0100 RST 2011
	fmt.Printf("%02d.%02d.%4d\n", t.Day(), t.Month(), t.Year())
	// 21.12.2011
	t = time.Now().UTC()
	fmt.Println(t) // Wed Dec 21 08:52:14 +0000 UTC 2011
	fmt.Println(time.Now()) // Wed Dec 21 09:52:14 +0100 RST 2011
	// calculating times:
	week = 60 * 60 * 24 * 7 * 1e9 // must be in nanosec
	week_from_now := t.Add(week)
	fmt.Println(week_from_now) // Wed Dec 28 08:52:14 +0000 UTC 2011
	// formatting times:
	fmt.Println(t.Format(time.RFC822)) // 21 Dec 11 0852 UTC
	fmt.Println(t.Format(time.ANSIC)) // Wed Dec 21 08:56:34 2011
	fmt.Println(t.Format("02 Jan 2006 15:04")) // 21 Dec 2011 08:52
	s := t.Format("20060102")
	fmt.Println(t, "=>", s)
	// Wed Dec 21 08:52:14 +0000 UTC 2011 => 20111221
}
```

## 指针
* `var intP *int; intP = &i1;` `intP` 存储了 `i1` 的内存地址；它指向了 `i1` 的位置，它引用了变量 `i1`。
* 一个指针变量可以指向任何一个值的内存地址。
* 使用一个指针引用一个值被称为间接引用；你可以在指针类型前面加上 `*` 号（前缀）来获取指针所指向的内容，这里的 `*` 号是一个类型更改器。
* 当一个指针被定义后没有分配到任何变量时，它的值为 nil。
* 符号 * 可以放在一个指针前，如 `*` intP，那么它将得到这个指针指向地址上所存储的值；这被称为反引用（或者内容或者间接引用）操作符；另一种说法是指针转移。
* 对于任何一个变量 `var`， 如下表达式都是正确的：`var == *(&var)`。
* 指针的一个高级应用是你可以传递一个变量的引用（如函数的参数），这样不会传递变量的拷贝；指针传递是很廉价的，只占用 4 个或 8 个字节；当程序在工作中需要占用大量的内存，或很多变量，或者两者都有，使用指针会减少内存占用和提高效率。
* 对一个空指针的反向引用是不合法的，并且会使程序崩溃；注意不是编译错误。

## 控制结构
### if-else
```go
if condition1 {
	// do something	
} else if condition2 {
	// do something else	
} else {
	// catch-all or default
}

if initialization; condition {
	// do something
}
```
### comma,ok 模式（pattern）
程序应该在最接近的位置检查所有相关的错误，至少需要暗示用户有错误发生并对函数进行返回，甚至中断程序。
```go
value, err := pack1.Function1(param1)
if err != nil {
	fmt.Printf("An error occured in pack1.Function1 with parameter %v", param1)
	return err
}
// 未发生错误，继续执行：
// Or
if err != nil {
	fmt.Printf("Program stopping with error %v", err)
	os.Exit(1)
}
// Or
if err := file.Chmod(0664); err != nil {
	fmt.Println(err)
	return err
}
// Or ok-pattern 
if value, ok := readData(); ok {
…
}
```
当打印到控制台时，可以将该函数返回的错误忽略；但当输出到文件流、网络流等具有不确定因素的输出对象时，应该始终检查是否有错误发生。
### switch 结构
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
变量 var1 可以是任何类型，而 val1 和 val2 则可以是同类型的任意值。类型不被局限于常量或整数，但必须是相同的类型；或者最终结果为相同类型的表达式。前花括号 { 必须和 switch 关键字在同一行。

您可以同时测试多个可能符合条件的值，使用逗号分割它们，例如：case val1, val2, val3。

每一个 case 分支都是唯一的，从上至下逐一测试，直到匹配为止。（ Go 语言使用快速的查找算法来测试 switch 条件与 case 分支的匹配情况，直到算法匹配到某个 case 或者进入 default 条件为止。）

一旦成功地匹配到某个分支，在执行完相应代码后就会退出整个 switch 代码块，也就是说您不需要特别使用 break 语句来表示结束。

因此，程序也不会自动地去执行下一个分支的代码。如果在执行完每个分支的代码后，还希望继续执行后续分支的代码，可以使用 fallthrough 关键字来达到目的。

### for 结构

* 基于计数器的迭代`for 初始化语句; 条件语句; 修饰语句 {}`；特别注意，永远不要在循环体内修改计数器，这在任何语言中都是非常差的实践！尽量在修饰语句修改计数器！
* 基于条件判断的迭代`for 条件语句 {}`。
* 无限循环`for { }`，无限循环的经典应用是服务器，用于不断等待和接受新的请求。
* `for-range` 结构可以迭代任何一个集合并可以获得每次迭代所对应的索引`for ix, val := range coll { }`；要注意的是，val 始终为集合中对应索引的值拷贝，因此它一般只具有只读性质，对它所做的任何修改都不会影响到集合中原有的值（译者注：如果 val 为指针，则会产生指针的拷贝，依旧可以修改集合中的原值）。

## 函数
* 简单的 return 语句也可以用来结束 for 死循环，或者结束一个协程（goroutine）。
* Go 里面有三种类型的函数：
	* 普通的带有名字的函数
	* 匿名函数或者lambda函数
	* 方法Methods
* 除了main()、init()函数外，其它所有类型的函数都可以有参数与返回值。
* 函数参数、返回值以及它们的类型被统称为函数签名。
* 函数能够接收参数供自己使用，也可以返回零个或多个值（我们通常把返回多个值称为返回一组值）；任何一个有返回值（单个或多个）的函数都必须以 return 或 panic 结尾。
* Go 默认使用按值传递来传递参数，也就是传递参数的副本。
* 如果你希望函数可以直接修改参数的值，而不是对参数的副本进行操作，你需要将参数的地址（变量名前面添加&符号，比如 `&variable`）传递给函数，这就是按引用传递，比如 `Function(&arg1)`，此时传递给函数的是一个指针。如果传递给函数的是一个指针，指针的值（一个地址）会被复制，但指针的值所指向的地址上的值不会被复制；我们可以通过这个指针的值来修改这个值所指向的地址上的值。（译者注：指针也是变量类型，有自己的地址和值，通常指针的值指向一个变量的地址。所以，按引用传递也是按值传递。）
* 几乎在任何情况下，传递指针（一个32位或者64位的值）的消耗都比传递副本来得少。
* 在函数调用时，像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）。
* 尽量使用命名返回值：会使代码更清晰、更简短，同时更加容易读懂；即使只有一个命名返回值，函数签名上面也需要使用 () 括起来。
* 传递指针给函数不但可以节省内存（因为没有复制变量的值），而且赋予了函数直接修改外部变量的能力，所以被修改的变量不再需要使用 return 返回。
* 我们要十分小心那些可以改变外部变量的函数，在必要时，需要添加注释以便其他人能够更加清楚的知道函数里面到底发生了什么。

### 变长参数

* 如果函数的最后一个参数是采用 ...type 的形式，那么这个函数就可以处理一个变长的参数，这个长度可以为 0，这样的函数称为变参函数。
* `func myFunc(a, b, arg ...int) {}` 这个函数接受一个类似某个类型的 slice 的参数。
* 如果参数被存储在一个 slice 类型的变量 slice 中，则可以通过 slice... 的形式来传递参数，`func(slice...)`调用变参函数。
* 变长参数可以作为对应类型的 slice 进行二次传递；一个接受变长参数的函数可以将这个参数作为其它函数的参数进行传递：
```go
func F1(s ...string) {
	F2(s...)
	F3(s)
}

func F2(s ...string) { }
func F3(s []string) { }
```
* 如果变长参数的类型并不是都相同的，有 2 种方案可以解决这个问题：
	* 使用结构，定义一个结构类型，假设它叫 Options，用以存储所有可能的参数：
```go
type Options struct {
	par1 type1,
	par2 type2,
	...
}
```
	* 使用空接口，该方案不仅可以用于长度未知的参数，还可以用于任何不确定类型的参数。
```go
func typecheck(..,..,values … interface{}) {
	for _, value := range values {
		switch v := value.(type) {
			case int: …
			case float: …
			case string: …
			case bool: …
			default: …
		}
	}
}
```

### defer和追踪

* 关键字 defer 允许我们推迟到函数返回之前（或任意位置执行 return 语句之后）一刻才执行某个语句或函数（为什么要在返回之后才执行这些语句？因为 return 语句同样可以包含一些操作，而不是单纯地返回某个值）。
* 关键字 defer 的用法类似于面向对象编程语言 Java 和 C# 的 finally 语句块，它一般用于释放某些已分配的资源。
* 当有多个 defer 行为被注册时，它们会以逆序执行（类似栈，即后进先出）。
* defer可以接受参数并使用当初的参数值。
```go
func f() {
	for i := 0; i < 5; i++ {
		defer fmt.Printf("%d ", i)
	}
}
// 输出 4 3 2 1 0
```
* 关键字 defer 允许我们进行一些函数执行完成后的收尾工作，例如：
	* 关闭文件流 
	* 解锁一个加锁的资源 
	* 打印最终报告
	* 关闭数据库链接
```go
package main

import "fmt"

func main() {
	doDBOperations()
}

func connectToDB() {
	fmt.Println("ok, connected to db")
}

func disconnectFromDB() {
	fmt.Println("ok, disconnected from db")
}

func doDBOperations() {
	connectToDB()
	fmt.Println("Defering the database disconnect.")
	defer disconnectFromDB() //function called here with defer
	fmt.Println("Doing some DB operations ...")
	fmt.Println("Oops! some crash or network error ...")
	fmt.Println("Returning from function here!")
	return //terminate the program
	// deferred function executed here just before actually returning, even if
	// there is a return or abnormal termination before
}

// ok, connected to db
// Defering the database disconnect.
// Doing some DB operations ...
// Oops! some crash or network error ...
// Returning from function here!
// ok, disconnected from db
```
* 使用 defer 语句实现代码追踪，一个基础但十分实用的实现代码执行追踪的方案就是在进入和离开某个函数打印相关的消息，即可以提炼为下面两个函数：
```go
func trace(s string) { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }
```
```go
package main

import "fmt"

func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

func a() {
	trace("a")
	defer untrace("a")
	fmt.Println("in a")
}

func b() {
	trace("b")
	defer untrace("b")
	fmt.Println("in b")
	a()
}

func main() {
	b()
}

//  entering: b
// in b
// entering: a
// in a
// leaving: a
// leaving: b
```
```go
// 上面代码的更简便的实现版本
package main

import "fmt"

func trace(s string) string {
	fmt.Println("entering:", s)
	return s
}

func un(s string) {
	fmt.Println("leaving:", s)
}

func a() {
	defer un(trace("a"))
	fmt.Println("in a")
}

func b() {
	defer un(trace("b"))
	fmt.Println("in b")
	a()
}

func main() {
	b()
}
```
* 使用 defer 语句来记录函数的参数与返回值(重点是可以记录返回值)
```go
package main

import (
	"io"
	"log"
)

func func1(s string) (n int, err error) {
	defer func() {
		log.Printf("func1(%q) = %d, %v", s, n, err)
	}()
	return 7, io.EOF
}

func main() {
	func1("Go")
}

// Output: 2011/10/04 10:46:11 func1("Go") = 7, EOF
```

### 递归函数
* 当一个函数在其函数体内调用自身，则称之为递归。
* 许多问题都可以使用优雅的递归来解决，比如说著名的快速排序算法。
* 在使用递归函数时经常会遇到的一个重要问题就是栈溢出：一般出现在大量的递归调用导致的程序栈内存分配耗尽，这个问题可以通过一个名为[懒惰求值](https://zh.wikipedia.org/wiki/%E6%83%B0%E6%80%A7%E6%B1%82%E5%80%BC)的技术解决。
* Go 语言中也可以使用相互调用的递归函数：多个函数之间相互调用形成闭环；因为 Go 语言编译器的特殊性，这些函数的声明顺序可以是任意的。

### 匿名函数和闭包
* 当我们不希望给函数起名字的时候，可以使用匿名函数，例如：`func(x, y int) int { return x + y }`。
* 匿名函数不能够独立存在（编译器会返回错误：`non-declaration statement outside function body`），但可以被赋值于某个变量，即保存函数的地址到变量中：`fplus := func(x, y int) int { return x + y }`，然后通过变量名对函数进行调用：`fplus(3,4)`。
* 也可以直接对匿名函数进行调用：`func(x, y int) int { return x + y } (3, 4)`。
* 匿名函数同样被称之为闭包（函数式语言的术语）：它们被允许调用定义在其它环境下的变量。闭包可使得某个函数捕捉到一些外部状态，例如：函数被创建时的状态。另一种表示方式为：一个闭包继承了函数所声明时的作用域。这种状态（作用域内的变量）都被共享到闭包的环境中，因此这些变量可以在闭包中被操作，直到被销毁。闭包经常被用作包装函数：它们会预先定义好 1 个或多个参数以用于包装。另一个不错的应用就是使用闭包来完成更加简洁的错误检查。
* 闭包函数保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量。
```go
package main

import "fmt"

func main() {
	var f = Adder()
	fmt.Print(f(1), " - ")
	fmt.Print(f(20), " - ")
	fmt.Print(f(300))
}

func Adder() func(int) int {
	var x int
	return func(delta int) int {
		x += delta
		return x
	}
}
// Output:1 - 21 - 321
```
* 在闭包中使用到的变量可以是在闭包函数体内声明的，也可以是在外部函数声明的：
```go
var g int
go func(i int) {
	s := 0
	for j := 0; j < i; j++ { s += j }
	g = s
}(1000) // Passes argument 1000 to the function literal.
```
* 一个返回值为另一个函数的函数可以被称之为工厂函数，这在您需要创建一系列相似的函数的时候非常有用：书写一个工厂函数而不是针对每种情况都书写一个函数；下面的函数演示了如何动态返回追加后缀的函数：
```go
func MakeAddSuffix(suffix string) func(string) string {
	return func(name string) string {
		if !strings.HasSuffix(name, suffix) {
			return name + suffix
		}
		return name
	}
}
// 现在，我们可以生成如下函数：
addBmp := MakeAddSuffix(".bmp")
addJpeg := MakeAddSuffix(".jpeg")
// 然后调用它们：
addBmp("file") // returns: file.bmp
addJpeg("file") // returns: file.jpeg
```
* 可以返回其它函数的函数和接受其它函数作为参数的函数均被称之为高阶函数，是函数式语言的特点。
* 能够知道一个计算执行消耗的时间是非常有意义的，尤其是在对比和基准测试中。
```go
start := time.Now()
longCalculation()
end := time.Now()
delta := end.Sub(start)
fmt.Printf("longCalculation took this amount of time: %s\n", delta)
```
* 当在进行大量的计算时，提升性能最直接有效的一种方式就是避免重复计算；通过在内存中缓存和重复利用相同计算的结果，称之为内存缓存。



































