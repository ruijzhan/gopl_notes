# 第三章 基本数据

- Go 的数据类型分为四大类：基础类型，聚合类型，引用类型，接口类型

## 3.1 整数

- rune 类型是 unit32 类型的同义词。 byte 类型是 unit8 类型的同义词。

- int 和 int32 是不同类型，之间需显式转换

- 有符号类型的范围是 -2^(n-1) ~ 2^(n-1)-1, 无符号是 2^n -1

- 注意二元操作符的优先级

- 无符号整数往往只用于位运算符和特定算数运算符，却极少用于表示非负值。

## 3.2 浮点数

- 有两种浮点数： float32 和 float64

- 绝大多数情况下，应优先选用 float64

- Go 的math包定义了函数创建无穷 Inf 和 非数值 NaN

## 3.3 复数

- 有 complex64 和 complex128, 分别由 float32 和 float64组成

- math/cmplx 包提供了复数运算函数库

## 3.4 布尔值

- && 优先级比 || 高

## 3.5 字符串

- 字符串是**不可变**的**字节**序列

- len 函数返回字符串的字节数

- \+ 号连接两个字符串生成一个新字符串

- 字符串可以通过 == 比较

### 3.5.1 字符串字面量

- 原生的字符串字面量用反引号 \`......\` 表示

### 3.5.2 Unicode

- rune 类型作为 int32 类型的别名

### 3.5.3 UTF-8

- UTF-8 以字节为单位对 Unicode 码点做变长编码（1-4个字节），长度以最高位中连续1的个数表示。

- 变长编码字符串无法以下标n直接访问第n个字符

- 用 unicode/utf8 中的 RuneCountInString() 获取UTF-8字符数

- utf8.DecodeRuneInString(string) 返回解码的文字符号r和长度

- 用 idx, r = range s { n++ } 来隐式读取字符，获取长度

- 用 r := rune(s) 把UTF-8编码的字符串转为Unicode码点序列

- 把Unicode序列转换会string，会输出UTF-8编码拼接结果

### 3.5.4 字符串和字节slice

- 4 个特别重要的标准包：bytes, strings, strconv, unicode (看它们的文档)

- 用 b := []byte(s) 把字符串s转换为字节slice b

- 用 bytes.Buffer 提高追加效率

### 3.5.5 字符串和数字的相互转换

- 整数转换成字符串： fmt.Sprintf 或者 strconv.Itoa

- Atoi, ParseInt, ParseUint 解释表示整数的字符串

- fmt.Scanf 处理单行混合输入

## 3.6 常量

### 3.6.1 常量生成器 iota

### 3.6.2 无类型常量