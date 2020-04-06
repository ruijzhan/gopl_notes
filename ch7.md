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

- P154（比较无聊，省略）

## 7.10 类型断言

- 类型断言用来从它的操作数 x 中把具体类型 T 的值提取出来，写作: x.(T)。如果成功，返回 T 类型的值。类型断言失败时导致宕机

- 如果类型断言中的 T 是一个接口类型，那么检查的是 x 的动态值是否满足T。如果成功，返回的是T类型的接口值，但保留了其中的动态类型和值。所以 x 不能是空接口值。

- 很少从一个接口值向要求更宽松（方法更少）的类型做类型断言。

- 在两个结果的类型断言赋值表达式中，第二个 bool 类型返回值只是断言是否成功：

  ```go
  if w, ok : w.(io.Writer); ok {
      // use w
  }
  ```

## 7.11 使用类型断言来识别错误

- 不要通过匹配字符串来判断实例

- 定义新结构体和其中的字段描述错误信息，再用指针接收者实现 error 接口。

- 获得 err 后用类型断言判断错误类型。

- 但如果特殊类型 err 被 fmt.Errorf 合并到一个大字符串中，结构信息就会丢失。所以错误识别通常在失败操作是马上处理，而不是返回给调用者。

## 7.12 通过接口类型断言来查询特性

- 声明一个局部 interface 类型 T，包含想要查询操作数 x 是否支持的方法。

- 用类型断言 if sw, ok := x.(T); ok { //调用所支持的方法 }

- 如果不 ok，则调用默认方法

## 7.13 类型分支

- 表面上 x 的类型是 interface{}，实际上是类型分支中类型的**联合识别**

  ```go
  func f(x interface {}) string{
      switch x:=x.(type) {   // 新声明的 x 不与外层词法块的 x 冲突
          case nil:
              return "NULL"
          case int, uint:
              return fmt.Sprintf("%d", x)
          case bool:
              if x {
                  return "TRUE"
              }
              return "FALSE"
          case string:
              return x
          default:
              panic(/*...*/)
      }
  }
  ```

## 7.14 示例：基于标记的XML解析

- 跟类型分支差不多。见P166

## 7.15 一些建议

- 不需要先设计接口再实现满足接口的类型。应先实现各种类型，在用接口抽象它们的行为。

- 如果接口和类型实现出于依赖的原因不能放在同一个包内，也可以用一个接口可以一个实现来解耦两个包

- 设计方法较少容易满足的小接口

- 不是所有的东西都一定是对象，全局函数，不完全封装的数据类型都应该有其位置
