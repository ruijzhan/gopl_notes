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

## 9.6 竞态检测器

## 9.7 示例：并发非阻塞缓存

## 9.8 gorouting 与线程

### 9.8.1 可增长的栈

### 9.8.2 gorouting 调度

### 9.8.3 GOMAXPROCS

### 9.8.4 gorouting 没有标识
