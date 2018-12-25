# Golang设计模式

Go语言惯用的设计和应用模式的精选集合。

## 创建模式

### 抽象工厂模式

提供一个用于创建一系列相关对象的接口

### 构建器模式

构建器模式将复杂对象的构造及其表示分开，以便相同的构造过程可以创建不同的表示。
在Go中，通常使用配置结构来实现相同的行为，但是将结构体传递给构建器方法会使用样板`if cfg.Field != nil {...}`检查填充代码 。
* *Implementation*

```go
package car

type Speed float64

const (
    MPH Speed = 1
    KPH       = 1.60934
)

type Color string

const (
    BlueColor  Color = "blue"
    GreenColor       = "green"
    RedColor         = "red"
)

type Wheels string

const (
    SportsWheels Wheels = "sports"
    SteelWheels         = "steel"
)

type Builder interface {
    Color(Color) Builder
    Wheels(Wheels) Builder
    TopSpeed(Speed) Builder
    Build() Interface
}

type Interface interface {
    Drive() error
    Stop() error
}
```

* *Usage*
```go
assembly := car.NewBuilder().Paint(car.RedColor)

familyCar := assembly.Wheels(car.SportsWheels).TopSpeed(50 * car.MPH).Build()
familyCar.Drive()

sportsCar := assembly.Wheels(car.SteelWheels).TopSpeed(150 * car.MPH).Build()
sportsCar.Drive()
```

### 工厂方法模式

工厂方法创建设计模式允许创建对象，而无需指定将要创建的对象的确切类型。

* *Implementation*
示例实现显示了如何为数据存储提供不同的后端，例如内存、磁盘存储。

```go
package data

import "io"

type Store interface {
    Open(string) (io.ReadWriteCloser, error)
}

// OR

package data

type StorageType int

const (
    DiskStorage StorageType = 1 << iota
    TempStorage
    MemoryStorage
)

func NewStore(t StorageType) Store {
    switch t {
    case MemoryStorage:
        return newMemoryStorage( /*...*/ )
    case DiskStorage:
        return newDiskStorage( /*...*/ )
    default:
        return newTempStorage( /*...*/ )
    }
}
```

* *Usage*
使用工厂方法，用户可以指定他们想要的存储类型。

```go
s, _ := data.NewStore(data.MemoryStorage)
f, _ := s.Open("file")

n, _ := f.Write([]byte("data"))
defer f.Close()
```

### 对象池模式

实例化并维护一组相同类型的对象实例，对象池创建设计模式用于根据需求期望准备和保留多个实例。

* *Implementation*
```go
package pool

type Pool chan *Object

func New(total int) *Pool {
    p := make(Pool, total)

    for i := 0; i < total; i++ {
        p <- new(Object)
    }

    return &p
}
```

* *Usage*
下面给出了一个对象池上的简单生命周期示例。

```go
p := pool.New(2)

select {
case obj := <-p:
    obj.Do( /*...*/ )

    p <- obj
default:
    // No more objects left — retry later or fail
    return
}
```

* *经验法则*
	* 在对象初始化比对象维护更昂贵的情况下，对象池模式很有用。
	* 如果需求激增而不是稳定需求，则维护开销可能会超过对象池的优势。
	* 由于预先初始化对象，它对性能有积极影响。

### 单例模式Singleton

Singleton创建设计模式将类型的实例化限制为单个对象。

* *Implementation*
```go
package singleton

type singleton map[string]string

var (
    once sync.Once

    instance singleton
)

func New() singleton {
    once.Do(func() {
        instance = make(singleton)
    })

    return instance
}
```

* *Usage*

```go
s := singleton.New()

s["this"] = "that"

s2 := singleton.New()

fmt.Println("This is ", s2["this"])
// This is that
```

* *经验法则*

单例模式表示全局状态，大多数情况下会降低可测试性。


































