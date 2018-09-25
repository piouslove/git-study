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

## 数组和切片
* 数组是具有相同 唯一类型 的一组已编号且长度固定的数据项序列（这是一种同构的数据结构）；数组长度也是数组类型的一部分。`var identifier [len]type`
* 如果我们想让数组元素类型为任意类型的话可以使用空接口作为类型；当使用值时我们必须先做一个类型判断。
* 对索引项为 i 的数组元素赋值可以这么操作：arr[i] = value，所以数组是可变的。
* 把一个大数组传递给函数会消耗很多内存。有两种方法可以避免这种现象：
	* 传递数组的指针
	* 使用数组的切片，在Go中通常使用切片
* 切片（slice）是对数组一个连续片段的引用（该数组我们称之为相关数组，通常是匿名的），所以切片是一个引用类型。
* 多个切片如果表示同一个数组的片段，它们可以共享数据；因此一个切片和相关数组的其他切片是共享存储的，相反，不同的数组总是代表不同的存储。数组实际上是切片的构建块。
* 因为切片是引用，所以它们不需要使用额外的内存并且比使用数组更有效率，所以在 Go 代码中 切片比数组更常用。
* 声明切片的格式是： `var identifier []type`（不需要说明长度）；一个切片在未初始化之前默认为 nil，长度为 0。
* 用切片组成的切片仍然指向相同的相关数组。
* 切片左闭右开，`arr1[start:end]`表示由数组 arr1 从 start 索引到 end-1 索引之间的元素构成的子集。
* 切片在内存中的组织方式实际上是一个有 3 个域的结构体：指向相关数组的指针，切片长度以及切片容量；这样就能确定一个切片。
* 绝对不要用指针指向 slice；切片本身已经是一个引用类型，所以它本身就是一个指针!!
* 因为字符串是纯粹不可变的字节数组，它们也可以被切分成 切片。
* `make`和`new`都在堆上分配内存，但是它们的行为不同，适用于不同的类型；new 函数分配内存，make 函数初始化。
	* `new(T)` 为每个新的类型T分配一片内存，初始化为 0 并且返回类型为`*T`的内存地址：这种方法 返回一个指向类型为 `T`，值为 0 的地址的指针，它适用于值类型如数组和结构体；它相当于 `&T{}`。
	* `make(T)` 返回一个类型为 T 的初始值，它只适用于3种内建的引用类型：切片、map 和 channel。
* 类型 []byte 的切片十分常见，Go 语言有一个 bytes 包专门用来解决这种类型的操作方法；它还包含一个十分有用的类型 Buffer:
```go
import "bytes"

type Buffer struct {
	...
}
```
* 读写长度未知的 bytes 最好使用 buffer，提供 Read 和 Write 方法。
* 通过 bytes.Buffer 串联字符串比使用 `+=` 要更节省内存和 CPU，尤其是要串联的字符串数目特别多的时候。
* 改变切片长度的过程称之为切片重组 reslicing，切片可以反复扩展直到占据整个相关数组。
* `func append(s[]T, x ...T) []T` 其中 `append` 方法将 0 个或多个具有相同类型 s 的元素追加到切片后面并且返回新的切片；追加的元素必须和原切片的元素同类型；如果 s 的容量不足以存储新增元素，append 会分配新的切片来保证已有切片元素和新增元素的存储；因此，返回的切片可能已经指向一个不同的相关数组了；`append` 方法总是返回成功，除非系统内存耗尽了；如果你想将切片 y 追加到切片 x 后面，只要将第二个参数扩展成一个列表即可：`x = append(x, y...)`。
* `func copy(dst, src []T) int` copy 方法将类型为 T 的切片从源地址 src 拷贝到目标地址 dst，覆盖 dst 的相关元素，并且返回拷贝的元素个数；源地址和目标地址可能会有重叠；拷贝个数是 src 和 dst 的长度最小值；如果 src 是字符串那么元素类型就是 byte；如果你还想继续使用 src，在拷贝结束后执行 `src = dst`。
* 字符串是由字节构建的，所以索引它们返回字节，而不是字符。字符串甚至可能不包含字符；事实上，“字符”的定义是不明确的，尝试通过定义字符串由字符组成来解决歧义是一个错误。
* 在内存中，一个字符串实际上是一个双字结构，即一个指向实际数据的指针和记录字符串长度的整数；因为指针对用户来说是完全不可见，因此我们可以依旧把字符串看做是一个值类型，也就是一个字符数组。
* 切片的底层指向一个数组，该数组的实际容量可能要大于切片所定义的容量；只有在没有任何切片指向的时候，底层的数组内存才会被释放，这种特性有时会导致程序占用多余的内存。
```go
// 函数 FindDigits 将一个文件加载到内存，然后搜索其中所有的数字并返回一个切片
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
// 这段代码可以顺利运行，但返回的 []byte 指向的底层是整个文件的数据。
// 只要该返回的切片不被释放，垃圾回收器就不能释放整个文件所占用的内存。
// 换句话说，一点点有用的数据却占用了整个文件的内存。
// 想要避免这个问题，可以通过拷贝我们需要的部分到一个新的切片中：
func FindDigits(filename string) []byte {
   b, _ := ioutil.ReadFile(filename)
   b = digitRegexp.Find(b)
   c := make([]byte, len(b))
   copy(c, b)
   return c
}
// 上面这段代码只能找到第一个匹配正则表达式的数字串；要想找到所有的数字，可以尝试下面这段代码：
func FindFileDigits(filename string) []byte {
   fileBytes, _ := ioutil.ReadFile(filename)
   b := digitRegexp.FindAll(fileBytes, len(fileBytes))
   c := make([]byte, 0)
   for _, bytes := range b {
      c = append(c, bytes...)
   }
   return c
}
```
* **[Golang字符串](https://blog.csdn.net/erlib/article/details/50907392)最全解释，必看**
* [字符串、数组和切片的应用](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/07.6.md)

## Map
* `var map1 map[keytype]valuetype` 在声明的时候不需要知道 map 的长度，map 是可以动态增长的；未初始化的 map 的值是 nil。
* key 可以是任意可以用 == 或者 != 操作符比较的类型，比如 string、int、float。所以数组、切片和结构体不能作为 key (译者注：含有数组切片的结构体不能作为 key，只包含内建类型的 struct 是可以作为 key 的），但是指针和接口类型可以；如果要用结构体作为 key 可以提供 Key() 和 Hash() 方法，这样可以通过结构体的域计算出唯一的数字或者字符串的 key。
* value 可以是任意类型的；通过使用空接口类型，我们可以存储任意值，但是使用这种类型作为值时需要先做一次类型断言。
* map 传递给函数的代价很小：在 32 位机器上占 4 个字节，64 位机器上占 8 个字节，无论实际上存储了多少数据。通过 key 在 map 中寻找值是很快的，比线性查找快得多，但是仍然比从数组和切片的索引中直接读取要慢 100 倍；所以如果你很在乎性能的话还是建议用切片来解决问题。
* 令 v := map1[key1] 可以将 key1 对应的值赋值给 v；如果 map 中没有 key1 存在，那么 v 将被赋值为 map1 的值类型的空值。
* 常用的 len(map1) 方法可以获得 map 中的 pair 数目，这个数目是可以伸缩的，因为 map-pairs 在运行时可以动态添加和删除。
* `myMap := make(map[keytype]valuetype, capacity)`出于性能的考虑，对于大的 map 或者会快速扩张的 map，即使只是大概知道容量，也最好先标明。
* 如果一个 key 要对应多个值，通过将 value 定义为 []int 类型或者其他类型的切片，就可以优雅的解决这个问题。
* 判断某个 key 是否存在我们可以这么用：`val1, isPresent = map1[key1]`；isPresent 返回一个 bool 值：如果 key1 存在于 map1，val1 就是 key1 对应的 value 值，并且 isPresent为true；如果 key1 不存在，val1 就是一个空值，并且 isPresent 会返回 false。
```go
_, ok := map1[key1] // 如果key1存在则ok == true，否则ok为false

// Or
if _, ok := map1[key1]; ok {
	// ...
}
```
* 从 map1 中删除 key1：直接 `delete(map1, key1)` 就可以；如果 key1 不存在，该操作不会产生错误。
* for-range 和 map 的配套用法
```go
for key, value := range map1 {
	...
}
// 如果你只关心值，可以这么使用
for _, value := range map1 {
	...
}
// 如果只想获取 key，你可以这么使用
for key := range map1 {
	fmt.Printf("key is: %d\n", key)
}
```
* 应当像 A 版本那样通过索引使用切片的 map 元素。在 B 版本中获得的项只是 map 值的一个拷贝而已，所以真正的 map 元素没有得到初始化。
```go
package main
import "fmt"

func main() {
	// Version A:
	items := make([]map[int]int, 5)
	for i:= range items {
		items[i] = make(map[int]int, 1)
		items[i][1] = 2
	}
	fmt.Printf("Version A: Value of items: %v\n", items)

	// Version B: NOT GOOD!
	items2 := make([]map[int]int, 5)
	for _, item := range items2 {
		item = make(map[int]int, 1) // item is only a copy of the slice element.
		item[1] = 2 // This 'item' will be lost on the next iteration.
	}
	fmt.Printf("Version B: Value of items: %v\n", items2)
}

// Output:
// Version A: Value of items: [map[1:2] map[1:2] map[1:2] map[1:2] map[1:2]]
// Version B: Value of items: [map[] map[] map[] map[] map[]]
```
* map 默认是无序的，不管是按照 key 还是按照 value 默认都不排序；如果你想为 map 排序，需要将 key（或者 value）拷贝到一个切片，再对切片排序。

## 包
* 像 fmt、os 等这样具有常用功能的内置包在 Go 语言中有 150 个以上，它们被称为标准库，大部分(一些底层的除外)内置于 Go 本身。

### 标准库
* `unsafe`: 包含了一些打破 Go 语言“类型安全”的命令，一般的程序中不会被使用，可用在 C/C++ 程序的调用中。
* `os`: 提供给我们一个平台无关性的操作系统功能接口，采用类Unix设计，隐藏了不同操作系统间差异，让不同的文件系统和操作系统对象表现一致。
* `os/exec`: 提供我们运行外部操作系统命令和程序的方式。
* `syscall`: 底层的外部包，提供了操作系统底层调用的基本接口。
* `archive/tar` 和 `/zip-compress`：压缩(解压缩)文件功能。
* `fmt`: 提供了格式化输入输出功能。
* `io`: 提供了基本输入输出功能，大多数是围绕系统功能的封装。
* `bufio`: 缓冲输入输出功能的封装。
* `path/filepath`: 用来操作在当前系统中的目标文件名路径。
* `flag`: 对命令行参数的操作。
* `strings`: 提供对字符串的操作。
* `strconv`: 提供将字符串转换为基础类型的功能。
* `unicode`: 为 unicode 型的字符串提供特殊的功能。
* `regexp`: 正则表达式功能。
* `bytes`: 提供对字符型分片的操作。
* `index/suffixarray`: 子字符串快速查询。
* `math`: 基本的数学函数。
* `math/rand`: 伪随机数生成。
* `sort`: 为数组排序和自定义集合。
* `math/big`: 大数的实现和计算。
* `container-/list-ring-heap`: 实现对集合的操作。
	* list: 双链表。
	* ring: 环形链表。
* `time`: 日期和时间的基本操作。
* `log`: 记录程序运行时产生的日志。
* `encoding/json`: 读取并解码和写入并编码 JSON 数据。
* `encoding/xml`:简单的 XML1.0 解析器。
* `text/template`:生成像 HTML 一样的数据与文本混合的数据驱动模板。
* `net`: 网络数据的基本操作。
* `http`: 提供了一个可扩展的 HTTP 服务器和基础客户端，解析 HTTP 请求和回复。
* `html`: HTML5 解析器。
* `runtime`: Go 程序运行时的交互操作，例如垃圾回收和协程创建。
* `reflect`: 实现通过程序运行时反射，让程序操作任意类型的变量。
* `exp` 包中有许多将被编译为新包的实验性的包。它们将成为独立的包在下次稳定版本发布的时候；如果前一个版本已经存在了，它们将被作为过时的包被回收。
* 通过使用 sync 包可以解决同一时间只能一个线程访问变量或 map 类型数据的问题；如果这种方式导致程序明显变慢或者引起其他问题，我们要重新思考来通过 goroutines 和 channels 来解决问题。

### 自定义包
* 按照惯例,子目录和包之间有着密切的联系：为了区分,不同包存放在不同的目录下，每个包(所有属于这个包中的 go 文件)都存放在和包名相同的子目录下。
* 当使用.来做为包的别名时，你可以不通过包名来使用其中的项目。`import . "./pack1"`
* `import _ "./pack1/pack1"` pack1包只导入其副作用，也就是说，只执行它的init函数并初始化其中的全局变量。
* 注释必须以 // 开始并无空行放在声明（包，类型，函数）前；godoc 会为每个文件生成一系列的网页。
* `godoc -http=:6060 -goroot="."` : `.` 是指当前目录，`-goroot` 参数可以是 `/path/to/my/package1` 这样的形式指出 package1 在你源码中的位置或接受用冒号形式分隔的路径，无根目录的路径为相对于当前目录的相对路径
* 自定义包的[目录结构](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/09.8.md)、go install 和 go test

## 结构体
* 结构体定义的一般方式如下：
```go
type identifier struct {
    field1 type1
    field2 type2
    ...
}
```
* 这样给它的字段赋值：
```go
var s T
s.a = 5
s.b = 8
```
* 点号在 Go 语言中这叫 选择器（selector）。无论变量是一个结构体类型还是一个结构体类型指针，都使用同样的 选择器符（selector-notation） 来引用结构体的字段。
```go
type myStruct struct { i int }
var v myStruct    // v是结构体类型变量
var p *myStruct   // p是指向一个结构体类型变量的指针
v.i
p.i
```
* 对于结构体来说，表达式 `new(Type)` 和 `&Type{}` 是等价的。
* Go 语言中，结构体和它所包含的数据在内存中是以连续块的形式存在的，即使结构体中嵌套有其他的结构体，这在性能上带来了很大的优势。
* 可以直接通过指针，像 `pers2.lastName="Woodward"` 这样给结构体字段赋值；也可以通过解指针的方式来设置值：`(*pers2).lastName = "Woodward"`。
* 结构体类型可以通过引用自身来定义；这在定义链表或二叉树的元素（通常叫节点）时特别有用，此时节点包含指向临近节点的链接（地址）。
* 将类型变成私有可以强制使用工厂方法创建结构体：
```go
type matrix struct {
    ...
}

func NewMatrix(params) *matrix {
    m := new(matrix) // 初始化 m
    return m
}
```
* 试图 `make()` 一个结构体变量，会引发一个编译错误，这还不是太糟糕，但是 `new()` 一个映射并试图使用数据填充它，将会引发运行时错误！ 因为 `new(Foo)` 返回的是一个指向 `nil` 的指针，它尚未被分配内存。所以在使用 `map` 时要特别谨慎。
* 结构体中的字段除了有名字和类型外，还可以有一个可选的标签（tag）：它是一个附属于字段的字符串，可以是文档或其他的重要标记；标签的内容不可以在一般的编程中使用，只有包 reflect 能获取它。
* 结构体可以包含一个或多个 匿名（或内嵌）字段，即这些字段没有显式的名字，只有字段的类型是必须的，此时类型就是字段的名字；匿名字段本身可以是一个结构体类型，即 结构体可以包含内嵌结构体。
* 通过类型 `outer.int` 的名字来获取存储在匿名字段中的数据，于是可以得出一个结论：在一个结构体中对于每一种数据类型只能有一个匿名字段。
* 同样地结构体也是一种数据类型，所以它也可以作为一个匿名字段来使用，内嵌结构体甚至可以来自其他包；内层结构体被简单的插入或者内嵌进外层结构体；这个简单的“继承”机制提供了一种方式，使得可以从另外一个或一些类型继承部分或全部实现。

### 方法
* 在 Go 语言中，结构体就像是类的一种简化形式，Go 方法是作用在接收者（receiver）上的一个函数，接收者是某种类型的变量；因此方法是一种特殊类型的函数。
* 接收者类型可以是（几乎）任何类型，不仅仅是结构体类型：任何类型都可以有方法，甚至可以是函数类型，可以是 int、bool、string 或数组的别名类型；但是接收者不能是一个接口类型，因为接口是一个抽象定义，但是方法却是具体实现；如果这样做会引发一个编译错误：invalid receiver type…。
* 最后接收者不能是一个指针类型，但是它可以是任何其他允许类型的指针。
* 一个类型加上它的方法等价于面向对象中的一个类；一个重要的区别是：在 Go 中，类型的代码和绑定在它上面的方法的代码可以不放置在一起，它们可以存在在不同的源文件，唯一的要求是：它们必须是同一个包的。
* 别名类型没有原始类型上已经定义过的方法。
* 定义方法的一般格式如下：`func (recv receiver_type) methodName(parameter_list) (return_value_list) { ... }`
* 如果 recv 是 receiver 的实例，Method1 是它的方法名，那么方法调用遵循传统的 `object.name` 选择器符号：`recv.Method1()`。
* 如果 recv 是一个指针，Go 会自动解引用。
* 接收者必须有一个显式的名字，这个名字必须在方法中被使用；如果方法不需要使用 recv 的值，可以用 _ 替换它，比如：`func (_ receiver_type) methodName(parameter_list) (return_value_list) { ... }`
* 类型和作用在它上面定义的方法必须在同一个包里定义，这就是为什么不能在 int、float 或类似这些的类型上定义方法；但是有一个间接的方式：可以先定义该类型（比如：int 或 float）的别名类型，然后再为别名类型定义方法。
* 在接收者是指针时，方法可以改变接收者的值（或状态），这点函数也可以做到（当参数作为指针传递，即通过引用调用时，函数也可以改变参数的状态）。
* 方法没有和数据定义（结构体）混在一起：它们是正交的类型；表示（数据）和行为（方法）是独立的。
* 鉴于性能的原因，recv 最常见的是一个指向 receiver_type 的指针（因为我们不想要一个实例的拷贝，如果按值调用的话就会是这样），特别是在 receiver 类型是结构体时，就更是如此了。
* 如果想要方法改变接收者的数据，就在接收者的指针类型上定义该方法。否则，就在普通的值类型上定义方法。
* 对于类型 `T`，如果在 `*T` 上存在方法 `Meth()`，并且 `t` 是这个类型的变量，那么 `t.Meth()` 会被自动转换为 `(&t).Meth()`；指针方法和值方法都可以在指针或非指针上被调用，GO会自动转换。
* 当一个匿名类型被内嵌在结构体中时，匿名类型的可见方法也同样被内嵌，这在效果上等同于外层类型 继承 了这些方法：将父类型放在子类型中来实现亚型。
* 可以覆写方法（像字段一样）：和内嵌类型方法具有同样名字的外层类型的方法会覆写内嵌类型对应的方法。
* 在 Go 语言中，通过在类型中嵌入所有必要的父类型，可以很简单的实现多重继承。
* 在编程中一些基本操作会一遍又一遍的出现，比如打开（Open）、关闭（Close）、读（Read）、写（Write）、排序（Sort）等等，并且它们都有一个大致的意思：打开（Open）可以作用于一个文件、一个网络连接、一个数据库连接等等；在 Go 语言中，通过使用接口，标准库广泛的应用了这些规则，在标准库中这些通用方法都有一致的名字，比如 Open()、Read()、Write()等；想写规范的 Go 程序，就应该遵守这些约定，给方法合适的名字和签名，就像那些通用方法那样，这样做会使 Go 开发的软件更加具有一致性和可读性；比如：如果需要一个 convert-to-string 方法，应该命名为 `String()`，而不是 `ToString()`。
* 在 Go 中，代码复用通过组合和委托实现，多态通过接口的使用来实现：有时这也叫 组件编程（Component Programming）。
* 如果真的需要更多面向对象的能力，看一下 [goop](https://github.com/lanl/goop) 包（Go Object-Oriented Programming），它由 Scott Pakin 编写: 它给 Go 提供了 JavaScript 风格的对象（基于原型的对象），并且支持多重继承和类型独立分派，通过它可以实现你喜欢的其他编程语言里的一些结构。
* 不要在 String() 方法里面调用涉及 String() 方法的方法，它会导致意料之外的错误，比如下面的例子，它导致了一个无限递归调用（TT.String() 调用 fmt.Sprintf，而 fmt.Sprintf 又会反过来调用 TT.String()...），很快就会导致内存溢出：
```go
type TT float64

func (t TT) String() string {
    return fmt.Sprintf("%v", t)
}
t.String()
```
* 如果类型定义了 String() 方法，它会被用在 fmt.Printf() 中生成默认的输出：等同于使用格式化描述符 %v 产生的输出；还有 fmt.Print() 和 fmt.Println() 也会自动使用 String() 方法。
* Go 开发者不需要写代码来释放程序中不再使用的变量和结构占用的内存，在 Go 运行时中有一个独立的进程，即垃圾收集器（GC），会处理这些事情，它搜索不再使用的变量然后释放它们的内存，可以通过 runtime 包访问 GC 进程。
* 通过调用 runtime.GC() 函数可以显式的触发 GC，但这只在某些罕见的场景下才有用，比如当内存资源不足时调用 runtime.GC()，它会在此函数执行的点上立即释放一大片内存，此时程序可能会有短时的性能下降（因为 GC 进程在执行）。
* 如果需要在一个对象 obj 被从内存移除前执行一些特殊操作，比如写到日志文件中，可以通过如下方式调用函数来实现：`runtime.SetFinalizer(obj, func(obj *typeObj))`；`func(obj *typeObj)`需要一个 `typeObj` 类型的指针参数 `obj`，特殊操作会在它上面执行；`func` 也可以是一个匿名函数；在对象被 GC 进程选中并从内存中移除以前，`SetFinalizer` 都不会执行，即使程序正常结束或者发生错误。


## 接口

* 接口定义了一组方法（方法集），但是这些方法不包含（实现）代码：它们没有被实现（它们是抽象的）。接口里也不能包含变量。
```go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```
* （按照约定，只包含一个方法的）接口的名字由方法名加 `[e]r` 后缀组成，例如 `Printer`、`Reader`、`Writer`、`Logger`、`Converter` 等等。还有一些不常用的方式（当后缀 `er` 不合适时），比如 `Recoverable`，此时接口名以 `able` 结尾，或者以 `I` 开头（像 .NET 或 Java 中那样）。
* Go 语言中的接口都很简短，通常它们会包含 0 个、最多 3 个方法。
* 类型不需要显式声明它实现了某个接口：接口被隐式地实现。多个类型可以实现同一个接口。
* 类型（比如结构体）实现接口方法集中的方法，每一个方法的实现说明了此方法是如何作用于该类型的：即实现接口，同时方法集也构成了该类型的接口。实现了 `Namer` 接口类型的变量可以赋值给 `ai` （接收者值），此时方法表中的指针会指向被实现的接口方法。当然如果另一个类型（也实现了该接口）的变量被赋值给 `ai`，这二者（译者注：指针和方法实现）也会随之改变。
* 实现某个接口的类型（除了实现接口方法外）可以有其他的方法；一个类型可以实现多个接口。
* 接口变量里包含了接收者实例的值和指向对应方法表的指针。
```go
package main

import "fmt"

type Shaper interface {
	Area() float32
}

type Square struct {
	side float32
}

func (sq *Square) Area() float32 {
	return sq.side * sq.side
}

type Rectangle struct {
	length, width float32
}

func (r Rectangle) Area() float32 {
	return r.length * r.width
}

func main() {

	r := Rectangle{5, 3} // Area() of Rectangle needs a value
	q := &Square{5}      // Area() of Square needs a pointer
	// shapes := []Shaper{Shaper(r), Shaper(q)}
	// or shorter
	shapes := []Shaper{r, q}
	fmt.Println("Looping through shapes for area ...")
	for n, _ := range shapes {
		fmt.Println("Shape details: ", shapes[n])
		fmt.Println("Area of this shape is: ", shapes[n].Area())
	}
}
```
* `io` 包里有一个接口类型 `Reader`:
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```
那么就可以写如下的代码：
```go
	var r io.Reader

	r = os.Stdin    // see 12.1
	r = bufio.NewReader(r)
	r = new(bytes.Buffer)
	f,_ := os.Open("test.txt")
	r = bufio.NewReader(f)
```
上面 `r` 右边的类型都实现了 `Read()` 方法，并且有相同的方法签名，`r` 的静态类型是 `io.Reader`。
* 一个接口可以包含一个或多个其他的接口，这相当于直接将这些内嵌接口的方法列举在外层接口中一样；比如接口 `File` 包含了 `ReadWrite` 和 `Lock` 的所有方法，它还额外有一个 `Close()` 方法。
```go
type ReadWrite interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
}

type Lock interface {
    Lock()
    Unlock()
}

type File interface {
    ReadWrite
    Lock
    Close()
}
```

### 类型断言
* 一个接口类型的变量 `varI` 中可以包含任何类型的值，必须有一种方式来检测它的 动态 类型，即运行时在变量中存储的值的实际类型；通常我们可以使用 类型断言 来测试在某个时刻 `varI` 是否包含类型 `T` 的值：
```go
v := varI.(T)       // unchecked type assertion
```
`varI` 必须是一个接口变量，否则编译器会报错：`invalid type assertion: varI.(T) (non-interface type (type of varI) on left)` 。
* 更安全的方式是使用以下形式来进行类型断言，应该总是使用下面的方式来进行类型断言；如果转换合法，`v` 是 `varI` 转换到类型 `T` 的值，`ok` 会是 `true`；否则 `v` 是类型 `T` 的零值，`ok` 是 `false`，也没有运行时错误发生。
```go
if v, ok := varI.(T); ok {  // checked type assertion
    Process(v)
    return
}
// varI is not of type T
```
* 接口变量的类型也可以使用一种特殊形式的 `switch` 来检测：`type-switch` ：
```go
switch t := areaIntf.(type) {
case *Square:
	fmt.Printf("Type Square %T with value %v\n", t, t)
case *Circle:
	fmt.Printf("Type Circle %T with value %v\n", t, t)
case nil:
	fmt.Printf("nil value: nothing to check?\n")
default:
	fmt.Printf("Unexpected type %T\n", t)
}
```
变量 `t` 得到了 `areaIntf` 的值和类型， 所有 `case` 语句中列举的类型（`nil` 除外）都必须实现对应的接口（在上例中即 `Shaper`），如果被检测类型没有在 `case` 语句列举的类型中，就会执行 `default` 语句。
* 可以用 `type-switch` 进行运行时类型分析，但是在 `type-switch` 不允许有 `fallthrough` 。
* 如果仅仅是测试变量的类型，不用它的值，那么就可以不需要赋值语句，比如：
```go
switch areaIntf.(type) {
case *Square:
	// TODO
case *Circle:
	// TODO
...
default:
	// TODO
}
```
* 在处理来自于外部的、类型未知的数据时，比如解析诸如 JSON 或 XML 编码的数据，类型测试和转换会非常有用。
* 假定 `v` 是一个值，然后我们想测试它是否实现了 `Stringer` 接口，可以这样做：
```go
type Stringer interface {
    String() string
}

if sv, ok := v.(Stringer); ok {
    fmt.Printf("v implements String(): %s\n", sv.String()) // note: sv, not v
}
```
* 接口是一种契约，实现类型必须满足它，它描述了类型的行为，规定类型可以做什么；接口彻底将类型能做什么，以及如何做分离开来，使得相同接口的变量在不同的时刻表现出不同的行为，这就是多态的本质。

### 方法与接口
* 在接口上调用方法时，必须有和方法定义时相同的接收者类型或者是可以从具体类型 `P` 直接可以辨识的：
	* 指针方法可以通过指针调用
	* 值方法可以通过值调用
	* 接收者是值的方法可以通过指针调用，因为指针会首先被解引用
	* 接收者是指针的方法不可以通过值调用，因为存储在接口中的值没有地址
* 将一个值赋值给一个接口时，编译器会确保所有可能的接口方法都可以在此值上被调用，因此不正确的赋值在编译期就会失败。
* Go 语言规范定义了接口方法集的调用规则：
	* 类型 `*T` 的可调用方法集包含接受者为 `*T` 或 `T` 的所有方法集
	* 类型 `T` 的可调用方法集包含接受者为 `T` 的所有方法
	* 类型 `T` 的可调用方法集不包含接受者为 `*T` 的方法

### 空接口
* 空接口或者最小接口 不包含任何方法，它对实现不做任何要求： `type Any interface {}` ；任何其他类型都实现了空接口（，`any` 或 `Any` 是空接口一个很好的别名或缩写。
* 可以给一个空接口类型的变量 `var val interface {}` 赋任何类型的值。
* 每个 `interface {}` 变量在内存中占据两个字长：一个用来存储它包含的类型，另一个用来存储它包含的数据或者指向数据的指针。
* 数据切片和空接口切片在内存中的布局不一样，不能直接赋值或复制，必须使用 for-range 语句来一个一个显式地复制：
```go
var dataSlice []myType = FuncReturnSlice()
var interfaceSlice []interface{} = make([]interface{}, len(dataSlice))
for i, d := range dataSlice {
    interfaceSlice[i] = d
}
```
* 一个接口的值可以赋值给另一个接口变量，只要底层类型实现了必要的方法；这个转换是在运行时进行检查的，转换失败会导致一个运行时错误：这是 Go 语言动态的一面。
* 

























