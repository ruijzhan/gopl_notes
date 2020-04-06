# 第七章 接口

Go中的接口类型是隐式实现。对于一个具体的类型，无须声明它实现了哪些接口，只需要提供接口所必须的方法。

## 7.1 接口即约定

- 参考 fmt.Fprintf 的实现

- **可取代性**：把一种类型替换为满足同一接口的另一种类型的特性

## 7.2 接口类型

- 可以通过组合已有接口得到新的接口:

  ```go
  package io
  
  type Reader interface {
      Read(p []byte) (n int, err error)
  }
  
  type Closer interface {
      Close() error
  }
  
  type ReaderCloser interface {
      Reader
      Closer
  }
  ```

## 7.3 实现接口

- 类型要实现接口，必须实现接口要求的所有方法

- 表达式实现了一个接口时，表达式才能赋值给接口

```go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = time.Second  // 错误
```

- 注意调用方法时的接收者变量指针隐式转换的语法糖，把变量赋值给接口值时无效。

- 空接口 interface{} 可以匹配任何类型。需要用类型断言将实际值还原。

- 当接口类型的方法暗示会修改接收者时（比如Write）方法，实现方法的接收者就应该为指针类型。

- Go 语言中需要时才定义新的抽象和分组，不用修改原有类型的定义

## 7.4 使用 flag.Value 来解析参数

- 实现 flag.Value 接口中的 String 和 Set 方法

- 见 P140 中的例子

## 7.5 接口值

- 接口值由具体的类型和该类型的一个值两部分组成。称为接口的**动态类型**和**动态值**

- 接口的零值是把它的动态类型和值都设置为 nil。可以用 == nil 来检测接口是否为零值。调用零值接口的方法会宕机。

- 赋值会把具体类型隐式转换为一个接口类型，改变接口类型的动态类型和值。用接口调用方法时，实际上是调用动态值上的方法。

- 接口值可以比较。如果都是nil，或者动态类型一致且动态值相等，那接口就是相等的。因为可以比较，所以可以作为map的键或者switch语句的操作数。

- 但比较两个接口时，如果动态值是不可比较的，例如 slice，会导致宕机

```go
var x interface{} = []int{1,2,3}
x == x  // 宕机
```

- 用 fmt 包的 %T 来获取接口的动态类型

- 注意含有空指针的非空接口：

  - 区别空的接口值和仅仅动态值为 nil 的接口值

  - 当把类型的空指针赋值给接口，接口的动态值为 nil， 在 != nil 时会true，导致接下来调用接口方法时引起宕机。

  - 解决方法是直接把接口值传递给接受接口值的函数，而不是传递满足接口的类型指针

## 7.6 使用 sort.Interface 来排序

- 要让一个类型可以被 sort.Sort() 排序，需要实现sort.Interface接口：

```go
package sort

type Interface interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}
```

- 用 sort.Strings(s []sting) 排序字符串 slice

- 要对一种集合类型按照不同规则排序，用这个类型作为底层类型声明新类型，用新类型实现自己的 sort.Interface 接口

## 7.7 http.Handler 接口

- 如何让一个类型多次实现一个接口中的同一个方法：（P151)

  - 类型中定义多个和接口的方法**相同签名**的方法

  - 另外定义一个**相同签名**函数类型

  - 在这个函数类型上定义接口要求的方法，用传入的参数调用接收者（接收者是一个函数）

  - 将类型定义的多个方法分别用函数类型完成类型转换，满足接口

## 7.8 error 接口

```go
type error interface {
    Error() string
}
```

```go
package errors

func New(text string) error {return &errorString{text}}

func errorString struct { text string }

func (e *errorString) Error() string {return e.text}
```

- 直接调用 errors.New 或者 fmt.Errorf 构造 error

- 通过满足 error 接口构建新的错误类型。例 P153

## 7.9 示例：表达式求值器

## 7.10 类型断言

## 7.11 使用类型断言来识别错误

## 7.12 通过接口类型断言来查询特性

## 7.13 类型分支

## 7.14 示例：基于标记的XML解析

## 7.15 一些建议
