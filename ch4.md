# 第四章 复合类型

## 4.1 数组

```go
var q [3]int = [3]int{1,2,3}
r := [...]int{1,2,3}
s := [...]{99: -1}
```

- 长度是数组类型的一部分，必须是常量表达式。

- 如果数组元素类型是可比较的，那数组也是可比较的

- 在函数调用中，数组作为参数是被看成**值传递**，所以要修改数组，需传入数组的地址

    ```go
    func zero(ptr *[32]byte) {
        for idx := range prt {    //for 获取 index
            ptr[idx] = 0
        }
    }
    //或者
    func zero(prt *[32]byte) {
        *ptr = [32]byte{}
    }
    ```

## 4.2 slice

- slice 有**指针**，**长度**，**容量**三个属性。用内置函数 len, cap 返回长度和容量

- 操作符 s[i:j] 创建一个新的slice。s可以是数组，数组指针或者 slice

- 如果对 slice s 的引用超过了其容量 cap(s)，会导致宕机。如果引用超过了长度，最终slice会变长

- 对字符串 s 做子串操作 s[i:j] 返回的是字符串，对slice s 做切片操作 s[i:j] 返回的是slice

- slice 包含所指向的数组的指针，将一个 slice 传递给函数时，在函数中可以修改底层数组的元素:

    ```go
    func reverse(s []int) {
        for i, j :=0, len(s) -1; i < j; i, j =i+1, j-1 {
            s[i], s[j] = s[j], s[i]
        }
    }

    a := [...]{1, 2, 3, 4, 5}
    reverse(a[:])     //转换为 slice ！！！
    fmt.Println(a)    // [5, 4, 3, 2, 1]
    ```

- 用 for range 循环遍历数组或者 slice：

    ```go
    for i := range sslice { /*只获取index*/}
    for i, e := range sslice { /* 获取index和元素 */}
    ```

- slice 的元素是非直接的，所以 slice **无法** 直接做比较，需要自己写函数来做比较。唯一运行 == 的操作是与 nil 比较。

- slice的零值是 nil。值为 nil 的 slice 表现与空 slice 一样。

- 检查一个slice 是非为空应用 len(s) == 0, 而不是 s == nil。因为 s 不为 nil的时候也可能为空。

- slice 中的元素会变，所以也不能用作 map 的 key。

- 对于引用类型，例如指针和通道，操作符 == 检查的是**引用相等性**

- make 可以创建一个具有指定元素类型，长度和容量的 slice。

    ```go
    make([]T, len)
    make([]T, len, cap)  //等同于 make([]T, cap)[:len]
    ```

### 4.2.1 append 函数

- 必须将 append 的调用结果再次赋值给传入 append 函数的 slice。

    ```go
    runes = append(runes, r)
    ```

- 任何可能改变 slice 的长度或者容量，或者使 slice 指向不同的底层数组，都需要更新 slice 变量。

### 4.2.2 slice 就地修改

- 使用内置的 copy 函数在两个 slice 之间拷贝元素

## 4.3 map

- 键的类型 K 必须是可以通过操作符 == 来比较的。但最好不要用浮点数

    ```go
    //空map
    ages := make(map[string]int)
    ages := map[string]int{}

    //字面量初始化
    ages := map[string]int{"foo": 21, "bar": 22}

    //用内置 delete 函数删除元素：
    delete(ages, "foo")
    ```

- 可以使用辅助函数把 slice 等不可直接比较的类型转换为 string，作为 map 的 key。间接实现不可比较作为 key 的功能

- map元素不是一个变量，不可以获取它的地址

- 使用 for range 循环遍历 map, 但元素顺序**不固定**。需显式给先给key排序再获取元素。

    ```go
    for key, value := range mmap { /*...*/ }
    // 或者
    for key := range mmap { /*...*/ }
    ```

- map类型的零值是nil，从零值map查找，删除元素，获取len，range循环和空map行为一致。但设置元素前，**必须初始化**

    ```go
    var ages map[string]int    //nil
    ```

- 通过下标访问map总是会有值，如果key不存在则返回值类型的空值。所以的第二个返回值来判断元素在不在map中：

    ```go
    if age, ok := ages["bob"]; !ok { /* 键 bob 不存在 */}
    ```

- map 类型之间不能比较, 但可以和nil比较。只能自己写函数遍历比较两个map, 注意区分“元素不存在”和“元素存在但值为零”的情况

- 可以用 map 实现 set 的部分功能。

- map 的值可以是任何类型。值可以再使用map类型实现延迟初始化

## 4.4 结构体

### 4.4.1 结构体字面量

### 4.4.2 结构体比较

### 4.4.3 结构体嵌套和匿名成员

## 4.5 JSON

## 4.6 文本和 HTML 模板