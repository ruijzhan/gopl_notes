# 第五章 函数

## 5.1 函数声明

- 返回值可以像形参一样命名，命名返回值会声明为一个局部变量，并初始化为零值

- 当两个函数拥有相同的形参列表和返回列表时，它们的函数类型或者**签名**是相同的。

- 形参变量都是函数的局部变量。

- 实参是按值传递的。如果实参是引用类型（指针，slice，map，函数或者通道），改变形参变量可以间接改变实参。

## 5.2 递归

```go
func visit(links []string, n *html.Node) []string {
    if n.Type == html.ElementNode && n.Data == "a" {
        for _, a := range n.Attr {
            if a.Key == "href" {
                links = append(links, a.Val)
            }
        }
    }
    for c := n.FirstChild; c != nil; c = c.NextSibling {
        links = visit(links, c)     //递归调用
    }
    return links
}
```

## 5.3 多值返回

- 裸返回：一个函数如果有命名的返回值，可以省略 return 的操作数

## 5.4 错误

- 如果错误只有一种情况，错误结果使用布尔值。如果有多种情况，使用 error 接口类型表示详细情况。

### 5.4.1 错误处理策略

- 第一种策略：将错误传递下去。可以新建一个 err，加入其它错误信息和老的 err 再 return。函数 f(x) 只负责报告函数 f 的行为和 x 的值。错误消息可能串联，避免首字母大写和换行。

```go
if err != nil {
    return nil, fmt.Errorf("parsing %s as HTML: %v", url, err)
}
```

- 第二种：出现不固定或者不可预测的错误时，短暂的间隔后重试，超出重试次数后再返回错误。

- 第三种：调用 os.Exit(1) 或者 log.Fatal("err message") 终止程序

- 第四种：只记录错误信息然后继续运行。log.Fprintf(os.Stderr, ...)

- 成功的逻辑一般不要放在 else 块内而是放在外层作用域

### 5.4.2 文件结束标识

```go
package os

import "errors"

var EOF = errors.New("EOF") //包级别变量
```

```go
for {
    _, _, err := in.ReadRune()
    if err == io.EOF {
        break  //读取结束
    }

}
```

## 5.5 函数变量

- 函数类型的零值是nil，调用一个空的函数变量会宕机。函数变量本身不可比较，但可以和空值比较。

```go
var f func(int) int
f(1)  // panic

if f != nil {
    f(1)
}
```

- 使用函数变量，可以在代码中将具体操作从代码执行逻辑中分开。不同的操作重用代码执行逻辑。

## 5.6 匿名函数

- func 关键字后没有函数的名称

- 在函数局部作用域内只能声明匿名函数。但是匿名函数可以赋值给变量再返回

- 匿名函数可以获得整个词法环境，使用外层函数中的变量。这样的函数变量是一个**闭包**

- **警告：捕获迭代变量**
  
  ```go
  var rmdirs []func()
  for _, dir := range tempDirs() {  // for 循环声明了 dir, 整个循环只声明了一次
      rmdirs = append(rmdirs, func() { os.RemoveAll(dir) })  //错误
  }
  ```

  - 在循环里创建的所有**函数变量**共享相同的 for 循环声明的变量，需要引入**内部变量**解决：

  ```go
  for _, dir := range tempDirs() {
      dir := dir  //为每次循环声明新的内部变量
      rmdirs = append(rmdirs, func() { os.RemoveAll(dir) })
  }

  ```

  - 在循环内调用匿名函数时，使用**显式参数**将迭代变量传给函数，以复制变量的副本

## 5.7 变长函数

```go
func sum(vals ...int) int {   //类型前加三个点
    r := 0
    for _, val := range vals {   //一个 slice
        r += val
    }
    return r
}

vals := []int{1, 2, 3, 4, 5}
fmt.Println(sum(vals...))  //加三个点把slice传给变长函数
```

## 5.8 函数延迟调用

- 可以有任意数量的 defer 语句, 执行时以调用defer语句的**倒序**执行

- defer 通常使用于成对的操作，比如打开和关闭，连接和断开，加锁和解锁。确定成功获得资源后，再执行defer调用

- 可以用defer在函数的入口和出口处设置调试行为：

  - 声明一个函数f() func()，执行在被调试函数入口时的操作。最后返回一个匿名函数变量，执行在出口的操作。

  - 在被调试函数开始时，defer f()()  //第二个（）是对返回的匿名函数的延迟调用

- 延迟执行的函数在 return 语句执行之后。而匿名函数可以获得外层函数中的变量（实参，命名结果变量），所以可以观察参数，修改返回结果。

## 5.9 宕机

- 碰到不可能发生的情况，宕机是最好的处理方式。

- panic 函数可以接受任何值作为参数

## 5.10 恢复

- 如果 recover 函数在在延迟函数的内部调用，而这个包含 defer 语句的函数发生 panic，recover会终止当前的宕机状态并返回宕机的值。

- 不应该尝试去恢复从另一个包内发生的宕机

- 从宕机中恢复的情况并不多，但可以通过一个明确的，非导出的类型作为宕机值。recover 中检查recover的返回值是否为这个特殊类型，是就另外处理。不是就继续宕机。
