# 第八章 gorouting和通道

- Go 有两种并发编程的风格：基于通道的 Communication Sequence Process 模式和共享内存的多线程模式 

## 8.1 gorouting

- 当 main 函数返回时，所有的 gorouting 都暴力地直接终止，然后程序退出

## 8.2 示例：并发时钟服务器

```go
// 创建监听对象
listenser, err := net.Listen("tcp", "localhost:8000")

// 阻塞直到收到请求，返回 net.Conn 实例
conn, err := listener.Accept()

// 用 gorouting 处理请求
go handleConn(conn)

// 向连接实例写入时间
_, err := io.WriteString(c, time.Now().Formart("15:04:05\n")

// 客户端连接服务器
conn, err := net.Dial("tcp","localhost:8000")

// 读取连接中的数据，写入stdout
io.Copy(os.Stdout, conn)
```

## 8.3 示例：并发回声服务器

- 一个连接的处理中，也可以使用多个 go 关键字：

  ```go
  input := bufio.NewScanner(conn)
  for input.Scan() {
      go echo(conn, input.Text(), 1 * time.Second)
  }
  ```

## 8.4 通道

- 用内置的 make 函数创建通道的**引用**：

  ```go
  var ch chan int          //零值为 nil
  ch == nil                //true
  ch = make(chan int)      // ch 的类型是 chan int, 无缓冲
  ch = make(chan int, 0)   // 无缓冲
  ch = make(chan int, 3)   // 缓冲容量为三
  p := new(chan int)       // *chan int
  *p == nil                // true
  p = &ch
  *p == ch                 // true，管道可以用 == 比较
  ```

- 管道可以发送，接收，关闭

  ```go
  ch <- x   //发送
  x = <- ch //接收
  <- ch     //接收并丢弃
  close(ch)
  ```

- 向已关闭的通道发送会宕机，试图关闭已关闭的管道也会宕机。读取已关闭的通道能获取已发送的值，直到为空。读取已关闭的空通道会立即获取通道元素对应类型的零值。

### 8.4.1 无缓冲通道

- 向无缓冲通道发送操作会被阻塞，直到另一个gorouting执行接收操作

- 相反，如果接收操作先执行，会被阻塞直到另一个gorouting执行发送操作

- 可以用 chan struct{} 类型的管道单纯进行同步：

  ```go
  // 声明管道
  done := make(chan struct{})
  
  // 发送端 gorouting
  done <- struct{}{}
  
  //接收端 gorouting
  <- done
  ```

### 8.4.2 管道

- 用读取通道返回的第二个 bool 类型的值判断是否读取的是已关闭的管道：

  ```go
  if x, ok := <- numbers; !ok {
    //通道 numbers 已关闭
  }
  ```

- 或者用 range 循环迭代读取管道，直到管道关闭

  ```go
  for x := range numbers {
    // ...
  }
  ```

- 只有在通知接收方数据都发送完毕时才需要关闭管道

### 8.4.3 单向通道类型

- 当一个通道用作函数的形参时，几乎总是被有意地限制不能发送或接收

- Go的类型系统提供了单向通道类型：

```go
chan<- int  //只能发送
<-chan int  //只能接收

func squarer(out chan<- int, in <-chan int) {
  for v := range in{
    out <- x*x
  }
  close(out)    //发送方才有义务关闭通道。关闭只读通道编译会报错
}
```

- 可以将双向通道转换为单向通道，反过来不行

### 8.4.4 缓冲通道

- 用 cap 函数获取通道的缓冲区容量，用 len 获取当前通道内的元素

- 不要用缓冲通道做队列

- 在多个 gorouting 向同一个无缓冲通道发送时，如果读只有一次，会卡住其余 gorouting 造成泄漏。应使用容量大于所有发送次数的缓冲通道。

## 8.5 并行循环

- 在循环中并行调用 gorouting 并向通道写入结果时时，应该：

  - 1，定义容量等于循环次数的通道，在循环外层从通道读取结果。
  
  - 2，定义无缓冲通道:
  
    - 使用 sync.WaitGroup 作为计数器，每次执行新 gorouting 前 wg.Add(1)

    - 每个gorouting 开始时 defer wg.Done()

    - 循环外开一个 gorouting 等待所有gorouting完成：wg.Wait()，然后关闭通道。

    - main gorouting 用range读取通道直到其关闭退出

## 8.6 示例：并发的 Web 爬虫

## 8.7 使用select多路复用

## 8.8 示例：并发目录遍历

## 8.9 取消

## 8.10 示例：聊天服务器
