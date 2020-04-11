# 第九章 使用共享变量实现并发

## 9.1 竞态

- 数据竞态发生于两个 gorouting 并发读写同一个变量并且至少一个是写入时

- 有三种方法来避免竞态：

  - 1，在并发的所有 gorouting 中不要修改变量。变量在并发开始前完成初始化

  - 2，变量的使用限制在单个 gorouting 内部。其他 gorouting 无法直接访问变量，必须通过通道发送读取和变更请求。

  - 3，使用互斥锁保护变量

## 9.2 互斥锁：sync.Mutex

```go
var (
    mu      sync.Mutex   // 未导出
    balance int
)

func Deposite(amount int) {
    mu.Lock()
    defer mu.Unlock()
    balance += amount
}
```

- 确保互斥量本身和被保护的变量未被导出

## 9.3 读写互斥锁：sync.RWMutex

```go
var (
    mu      sync.RWMutex
    balance int
)
func Balance() int {
    mu.RLock()          // 使用读锁。在写的函数中依然使用 mu.Lock()
    defer mu.RUnlock()
    return balance
}
```

## 9.4 内存同步

- 通道通信或者互斥锁操作这样的同步原语会导致处理器把积累的写操作刷回内存并提交

- 不同的 gorouting 共享变量可能读取到在CPU缓存中还未刷回内存的过期变量值

- 所以尽量把变量限制在单个 gorouting 中。如果实在不行，则使用互斥锁确保修改刷回内存。

## 9.5 延迟初始化：sync.Once

```go
func loadIcons() {
    //...
}

var loadIconsOnce sync.Once
var icons map[string]image.Image

func Icon(name string) image.Image {
    loadIconsOnce.Do(loadIcons)
    return icons[name]
}
```

- Once 包含一个布尔变量和互斥量。布尔变量记录是否初始化完成，互斥量保护初始化代码。Do函数接收初始化函数变量做参数。

## 9.6 竞态检测器

- 把 -race 命令行参数加到 go build, run, test 命令里启动该功能

## 9.7 示例：并发非阻塞缓存

- 在保存结果的结构体中嵌入一个通道。通过关闭通道来广播结果值准备好的事件。

## 9.8 gorouting 与线程

### 9.8.1 可增长的栈

- Go 中的栈大小不固定，可以根据需求正大和缩小。

### 9.8.2 gorouting 调度

- Go 调度器不需要切换到内核语境，所以调度 gorouting 比线程成本低

### 9.8.3 GOMAXPROCS

- Go 调度器使用 GOMAXPROCS 参数切断使用多少个OS线程同时执行 Go 代码。默认是机器上的 CPU 核数

- 可以设置 GOMAXPROCS 环境变量或者 runtime.GOMAXPROCS 来控制这个参数

### 9.8.4 gorouting 没有标识
