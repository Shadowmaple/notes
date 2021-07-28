# go 类型转换

```go
// int到string
string := strconv.Itoa(int)
// int64到string
string := strconv.FormatInt(int64, 10)
// uint64到string
string := strconv.FormatUint(uint64, 10)

// string到int
int, err := strconv.Atoi(string)
// string到int64
int64, err := strconv.ParseInt(string, 10, 64)
// string到uint64
uint64, err := strconv.ParseUint(string, 10, 64)
// 将字符串转换为 float64 型
strconv.ParseFloat(s string, bitSize int) (f float64, err error) 
```
# 字符串拼接
1. `fmt.Sprintf`，低效
2. \+ 号连接，低效
3. `strings.Join()`，高效
4. 字节缓冲（`bytes.Buffer`），更高效

# strings包
## 前缀和后缀
```go
// HasPrefix 判断字符串 s 是否以 prefix 开头
strings.HasPrefix(s, prefix string) bool

// HasSuffix 判断字符串 s 是否以 suffix 结尾
strings.HasSuffix(s, suffix string) bool

// Contains 判断字符串 s 是否包含 substr
strings.Contains(s, substr string) bool
```
## 判断子字符串或字符在父字符串中出现的位置（索引）
Index 返回字符串 str 在字符串 s 中的索引（str 的第一个字符的索引），-1 表示字符串 s 不包含字符串 str
```go
strings.Index(s, str string) int
```

LastIndex 返回字符串 str 在字符串 s 中最后出现位置的索引（str 的第一个字符的索引），-1 表示字符串 s 不包含字符串 str
```go
strings.LastIndex(s, str string) int
```
如果 ch 是非 ASCII 编码的字符，建议使用以下函数来对字符进行定位

```go
strings.IndexRune(s string, ch int) int
```
## 字符串替换
Replace 用于将字符串 str 中的前 n 个字符串 old 替换为字符串 new，并返回一个新的字符串，如果 n = -1 则替换所有字符串 old 为字符串 new

```go
strings.Replace(str, old, new, n) string
```

## 统计字符串出现次数
Count 用于计算字符串 str 在字符串 s 中出现的非重叠次数
```go
strings.Count(s, str string) int
```

## 重复字符串
Repeat 用于重复 count 次字符串 s 并返回一个新的字符串
```go
strings.Repeat(s, count int) string
```

## 修改字符串大小写
`ToLower` 将字符串中的 Unicode 字符全部转换为相应的小写字符

```go
strings.ToLower(s) string
```

`ToUpper` 将字符串中的 Unicode 字符全部转换为相应的大写字符

```go
strings.ToUpper(s) string
```

## 修剪字符串
可以使用 `strings.TrimSpace(s) `来剔除字符串开头和结尾的空白符号；如果想要剔除指定字符，则可以使用`strings.Trim(s, "cut")` 来将开头和结尾的 cut 去除掉。该函数的第二个参数可以包含任何字符，如果只想剔除开头或者结尾的字符串，则可以使用 `TrimLeft` 或者 `TrimRight` 来实现

## 分割字符串
`strings.Fields(s)` 将会利用 1 个或多个空白符号来作为动态长度的分隔符将字符串分割成若干小块，并返回一个 slice，如果字符串只包含空白符号，则返回一个长度为 0 的 slice

`strings.Split(s, sep)` 用于自定义分割符号来对指定字符串进行分割，同样返回 slice

## 拼接 slice 到字符串
Join 用于将元素类型为 string 的 slice 使用分割符号来拼接组成一个字符串

```go
Strings.Join(sl []string, sep string)
```

## 从字符串中读取内容
函数 strings.NewReader(str) 用于生成一个 Reader 并读取字符串中的内容，然后返回指向该 Reader 的指针，从其它类型读取内容的函数还有
Read() 从 []byte 中读取内容
ReadByte() 和 ReadRune() 从字符串中读取下一个 byte 或者 rune

# 字符类型判断
判断是否为**字母**：`unicode.IsLetter(ch)`

判断是否为**数字**：`unicode.IsDigit(ch)`

判断是否为**空白符号**：`unicode.IsSpace(ch)`

# 标准输入输出

```go
fmt.Print() // 不换行输出
fmt.Println() // 换行输出
fmt.Printf("%c",a) // 格式化输出

fmt.Scan(&a)
fmt.Scanf("%d",&a)
```
# go tool

```shell
go fmt      # 自动统一代码格式
go vet      # 检查语法错误
go env      # 查看当前go环境信息
go clean    # 清理我们编译生成的文件，比如生成的可执行文件，生成obj对象等
```
# defer
1.  defer后面必须跟函数，如 `defer fmt.Println("ok")`
2.  defer后面的函数在defer语句所在的函数结束时会被调用
3.  如果函数里面有多条defer指令，他们的执行顺序是反序，即后定义的defer先执行，栈的方式
5.  defer与return执行顺序：return后的语句先，defer后。

```go
func returnFunc() int {
    fmt.Println("return func called")
    return 0
}

func foo() int {
    defer func() {
        fmt.Println("defer func called")
    }()
    return returnFunc()
}

/*
结果：
returnFunc called
defer func called
*/
```

6.  虽然先return再defer，但defer会影响返回值。比如以下程序，res的值为1，而不是0。

```go
func foo() (res int) {
    defer func() {
    	res = 1
    }()
    return res
}
```

7.  defer 最大的功能是 panic 后依然有效，所以常用来捕获异常或关闭一些资源（比如锁、channel）

# new() 和 make() 的区别
看起来二者没有什么区别，都在堆上分配内存，但是它们的行为不同，适用于不同的类型。

`new(T)` 为每个新的类型 T 分配一片内存，初始化为 0 并且返回类型为*T的内存地址。这种方法返回一个指向类型为 T、值为 0 的指针，它适用于值类型，如数组和结构体；它相当于 `&T{}`。

`make(T)` 返回一个类型为 T 的初始值，它只适用于3种**内建的引用类型**：slice、map 和 channel

换言之，new 函数分配内存，make 函数初始化

# bytes.Buffer
它是一个长度可变的 bytes 的 buffer，提供 Read 和 Write 方法，因为读写长度未知的 bytes 最好使用 buffer。

Buffer 的定义方式：

```go
var buffer bytes.Buffer

// 或者使用 new 获得一个指针
var r *bytes.Buffer = new(bytes.Buffer)

// 或者通过函数
// 创建一个 Buffer 对象并且用 buf 初始化好
// NewBuffer 最好用在从 buf 读取的时候使用
func NewBuffer(buf []byte) *Buffer {
    // ...
}
```

通过 buffer 串联字符串：

创建一个 buffer，通过 `buffer.WriteString(s)` 方法将字符串 s 追加到后面，最后再通过 `buffer.String()` 方法转换为 string

# append()常见操作

1.  将切片 b 的元素追加到切片 a 之后：`a = append(a, b...)`

2.  复制切片 a 的元素到新的切片 b 上：

```go
b = make([]T, len(a))
copy(b, a)
```
3.  删除位于索引 i 的元素：`a = append(a[:i], a[i+1:]...)`
4.  切除切片 a 中从索引 i 至 j 位置的元素：`a = append(a[:i], a[j:]...)`
5.  为切片 a 扩展 j 个元素长度：`a = append(a, make([]T, j)...)`
6.  在索引 i 的位置插入元素 x：`a = append(a[:i], append([]T{x}, a[i:]...)...)`
7.  在索引 i 的位置插入长度为 j 的新切片：`a = append(a[:i], append(make([]T, j), a[i:]...)...)`
8.  在索引 i 的位置插入切片 b 的所有元素：`a = append(a[:i], append(b, a[i:]...)...)`
9.  取出位于切片 a 最末尾的元素 x：`x, a = a[len(a)-1], a[:len(a)-1]`
10.  将元素 x 追加到切片 a：`a = append(a, x)`

因此，可以使用切片和 append 操作来表示任意可变长度的序列。

从数学的角度来看，切片相当于向量，如果需要的话可以定义一个向量作为切片的别名来进行操作

# 接口

接口是一种契约，实现类型必须满足它，它描述了类型的行为，规定类型可以做什么。接口彻底将类型能做什么，以及如何做分离开来，使得相同接口的变量在不同的时刻表现出不同的行为，这就是多态的本质。

## 接口值

一个接口类型的值包含两个部分：**动态类型**和**动态值**

在比较两个接口时，若两个接口值的动态类型一致，但对应的动态值是不可比较的（如slice、map和函数），那么这个比较将会以崩溃的形式失败

所以在比较两个接口时，必须要先确认接口值包含的动态值可以比较。

注意：**含有空指针的非空接口**

```go
func main() {
    var buf *bytes.Buffer
    f(buf)
}

func f(out io.Writer) {
    if out != nil {
        out.Write([]byte("done")) // 宕机，对空指针取引用值
    }
}
```

上述的程序会产生崩溃，因为对于 buf ，它的动态类型是 `*bytes.Buffer`，所以它是一个非空接口，在比较`out != nil`时结果是true，但它的动态值为 nil ，所以它是一个**含有空指针的非空接口**，对空指针进行`Write`操作会panic。

## 方法集的调用规则

在接口上调用方法时，必须有和方法定义时相同的接收者类型或者是可以从具体类型 `P` 直接可以辨识的：

-   指针方法可以通过指针调用
-   值方法可以通过值调用
-   **接收者是值的方法可以通过指针调用，因为指针会首先被解引用**
-   **接收者是指针的方法不可以通过值调用，因为存储在接口中的值没有地址**

Go 语言规范定义了接口方法集的调用规则：

+   类型 *T 的可调用方法集包含接受者为 *T 或 T 的所有方法集
+   类型 T 的可调用方法集包含接受者为 T 的所有方法
+   类型 T 的可调用方法集不包含接受者为 *T 的方法

## 类型断言

类型断言的本质，跟类型转换类似，都是类型之间进行转换，不同之处在于，类型断言是在接口之间进行。

```go
var i interface{} = 44
// 直接断言若实现不了目标的类型就会panic
value := i.(int)
// 安全做法
value, ok := i.(int)
```

# 查看变量地址

使用`unsafe.Pointer()`

```go
num := 1
fmt.Println(unsafe.Pointer(&num))

// 输出：
// 0xc000344880
```



# 在线测试

[The go palyground](https://play.golang.org/)

[菜鸟工具](https://c.runoob.com/compile/21)

