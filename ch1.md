# 第一章 入门

## 1.1 hello, world

- 一个包由一个或多个 .go 文件组成，放在一个文件夹中

- 每一个源文件的开始都用 package 声明

- import 声明跟在 package 声明之后

- 用 gofmt -w file_name 或 goimport -w file_name 格式化代码文件

## 1.2 命令行参数

- os.Args 是一个字符串 slice

- slice s 用 s[i] 访问单个元素，s[m:n] 访问一段连续子区间，用 len(s) 表示长度

- var 关键字 **声明** 变量

- 变量可以被 **隐式** 的初始化为类型的空值

- += 是一个赋值操作符

- i++ 是语句，不是表达式。不能 j = i++

- for 是 Go 里面的 **唯一** 循环语句


```go
for init; condition; post {
    // ...
}

// 传统的 while 循环
for condition {
    // ...
}

// 传统的无限循环
for {
    // ...
}
```

- 用 **range** 每次迭代 slice 产生索引和元素，不需要的变量可以赋值给 _

```go
for _, arg := range os.Args[1:] {
    // ...
}
```

- 以下几种变量声明是等价的。实践中应使用前两种形式，

```go
s := ""              // 通常在函数内部使用，不适合包级别的变量。显示的初始化说明初始值的重要性
var s string         // 说明初始化值不重要
var s = ""           // 很少用，除非声明多个变量
var s string = ""    // 在类型不一致时是必须的
```

```go
fmt.Println(strings.Join(os.Args[1:], " ")) 

// 不关心格式
fmt.Println(os.Args[1:])
```

## 1.3 找出重复行

- 用 make 创建 map 数据结构的 **引用**。用 range 迭代 map，key 的顺序不固定

```go
mmap := make(map[int]string)
// ...
for key, val := range mmap {
    // ...
}
```

- fmt.Printf 函数常用的 **转义字符** (verb):

|verb|描述|
|-|-|
|%d | 十进制数 |
|%x, %o, %b|十六进制，八进制，二进制整数|
|%f| 浮点数|
|%t|布尔类型|
|%c|字符，Unicode码点（rune）|
|%s|字符串|
|%q|带引号的字符串("abc") 或者字符 ('c')
|%v| 内置格式的任何值|
|%T| 任何值的类型|
|%%| 百分号本身

## 1.4 GIF 动画

- const 声明可以在包级别或者函数内。常量必须是数字，字符串或布尔值。

## 1.5 获取一个URL

- 记得调用 resp.Body.Close() 避免资源泄露

## 1.6 并发获取多个 URL

- main 函数在一个 gorouting 中执行

- 通道：

```go
ch := make(chan string)  // 创建通道

ch <- expression         // 向通道发送值

<- ch                    // 从通道接收值
```

## 1.7 一个Web服务器

- 用 sync.Mutex 保护临界区域避免竞态

## 1.8 其他内容

- switch 语句：

```go
switch expression {  // 可以包含：一个短变量声明，递增或赋值语句，或者函数调用。
    case match1:
        // ...
    case match2:
        // ...
    default:
        // ...
}

switch {    //等价于 switch true
    case x > 0:
    // ...
}
```

- 用标签化语句让 break 和 continue 跳出多层嵌套循环
