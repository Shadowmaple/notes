# golang 要注意的坑

## 不能使用简短声明来设置字段的值

struct 的变量字段不能使用 `:=` 来赋值以使用预定义的变量来避免解决

```go
// 错误示例
type info struct {
	result int
}

func work() (int, error) {
	return 3, nil
}

func main() {
	var data info
	data.result, err := work()	// error: non-name data.result on left side of :=
	fmt.Printf("info: %+v\n", data)
}


// 正确示例
func main() {
	var data info
	var err error	// err 需要预声明

	data.result, err = work()
	if err != nil {
		fmt.Println(err)
		return
	}

	fmt.Printf("info: %+v\n", data)
}
```

## 显式类型的变量无法使用 nil 来初始化

`nil` 是 interface、function、pointer、map、slice 和 channel 类型变量的默认初始值。但声明时不指定类型，编译器也无法推断出变量的具体类型。

```go
// 错误示例
func main() {
    var x = nil	// error: use of untyped nil
	_ = x
}


// 正确示例
func main() {
	var x interface{} = nil
	_ = x
}
```

## 直接使用值为 nil 的 slice、map

允许对值为 nil 的 slice 添加元素，但对值为 nil 的 map 添加元素则会造成运行时 panic

```go
// map 错误示例
func main() {
    var m map[string]int
    m["one"] = 1		// error: panic: assignment to entry in nil map
    // m := make(map[string]int)// map 的正确声明，分配了实际的内存
}    


// slice 正确示例
func main() {
	var s []int
	s = append(s, 1)
}
```

## map 容量

在创建 map 类型的变量时可以指定容量，但不能像 slice 一样使用 `cap()` 来检测分配空间的大小：

```go
// 错误示例
func main() {
	m := make(map[string]int, 99)
	println(cap(m))
    // error: invalid argument m1 (type map[string]int) for cap
}
```

## string 类型的变量值不能为 nil

对那些喜欢用 `nil` 初始化字符串的人来说，这就是坑：

```go
// 错误示例
func main() {
	var s string = nil	// cannot use nil as type string in assignment
	if s == nil {	// invalid operation: s == nil (mismatched types string and nil)
		s = "default"
	}
}


// 正确示例
func main() {
	var s string	// 字符串类型的零值是空串 ""
	if s == "" {
		s = "default"
	}
}
```

## Array 类型的值作为函数参数

在 C/C++ 中，数组（名）是指针。将数组作为参数传进函数时，相当于传递了数组内存地址的引用，在函数内部会改变该数组的值。

在 Go 中，数组是值。作为参数传进函数时，传递的是数组的原始值拷贝，此时在函数内部是无法更新该数组的：

如果想修改参数数组：

-   直接传递指向这个数组的指针类型：
-   直接使用 slice：即使函数内部得到的是 slice 的值拷贝，但依旧会更新 slice 的原始数据（底层 array）

## 访问 map 中不存在的 key

访问map中不存在的key时，Go 会返回元素对应数据类型的零值，比如 `nil`、`''` 、`false` 和 0，取值操作总有值返回，故不能通过取出来的值来判断 key 是不是在 map 中。

检查 key 是否存在可以用 map 直接访问，检查返回的第二个参数即可：

```go
// 错误的 key 检测方式
func main() {
	x := map[string]string{"one": "2", "two": "", "three": "3"}
	if v := x["two"]; v == "" {
		fmt.Println("key two is no entry")
        // 键 two 存不存在都会返回的空字符串
	}
}

// 正确示例
func main() {
	x := map[string]string{"one": "2", "two": "", "three": "3"}
	if _, ok := x["two"]; !ok {
		fmt.Println("key two is no entry")
	}
}
```

## string 与 byte slice 之间的转换

当进行 string 和 byte slice 相互转换时，参与转换的是拷贝的原始值。这种转换的过程，与其他编程语的强制类型转换操作不同，也和新 slice 与旧 slice 共享底层数组不同。

Go 在 string 与 byte slice 相互转换上优化了两点，避免了额外的内存分配：

-   在 `map[string]` 中查找 key 时，使用了对应的 `[]byte`，避免做 `m[string(key)]` 的内存分配
-   使用 `for range` 迭代 string 转换为 []byte 的迭代：`for i,v := range []byte(str) {...}`

## string 与索引操作符

对字符串用索引访问返回的不是字符，而是一个 byte 值。

```go
func main() {
	x := "ascii"
	fmt.Println(x[0])		// 97
	fmt.Printf("%T\n", x[0])// uint8
}
```

如果需要使用 `for range` 迭代访问字符串中的字符（unicode code point / rune），标准库中有 `"unicode/utf8"` 包来做 UTF8 的相关解码编码。另外 [utf8string](https://link.juejin.im?target=https%3A%2F%2Fgodoc.org%2Fgolang.org%2Fx%2Fexp%2Futf8string) 也有像 `func (s *String) At(i int) rune` 等很方便的库函数。

## 字符串并不都是 UTF8 文本

string 的值不必是 UTF8 文本，可以包含任意的值。只有字符串是文字字面值时才是 UTF8 文本，字串可以通过转义来包含其他数据。

判断字符串是否是 UTF8 文本，可使用 "unicode/utf8" 包中的 `ValidString()` 函数：

```go
func main() {
	str1 := "ABC"
	fmt.Println(utf8.ValidString(str1))	// true

	str2 := "A\xfeC"
	fmt.Println(utf8.ValidString(str2))	// false

	str3 := "A\\xfeC"
	fmt.Println(utf8.ValidString(str3))	// true	// 把转义字符转义成字面值
}
```

## 字符串的长度

在 Python 中：

```python
data = u'♥'  
print(len(data)) # 1
```

然而在 Go 中：

```go
func main() {
	char := "♥"
	fmt.Println(len(char))	// 3
}
```

Go 的内建函数 `len()` 返回的是字符串的  byte 数量，而不是像 Python  中那样是计算 Unicode 字符数。

如果要得到字符串的字符数，可使用 "unicode/utf8" 包中的 `RuneCountInString(str string) (n int)`

```go
func main() {
	char := "♥"
	fmt.Println(utf8.RuneCountInString(char))	// 1
}
```

**注意：** `RuneCountInString` 并不总是返回我们看到的字符数，因为有的字符会占用 2 个 rune：

```go
func main() {
	char := "é"
	fmt.Println(len(char))	// 3
	fmt.Println(utf8.RuneCountInString(char))	// 2
	fmt.Println("cafe\u0301")	// café	// 法文的 cafe，实际上是两个 rune 的组合
}
```

## 在多行 array、slice、map 语句中缺少 `,` 号

```go
func main() {
	x := []int {
		1,
		2	// syntax error:unexpected newline, expecting comma or }
	}
    y := []int{1,2,}
	z := []int{1,2}	
}
```

声明语句中 `}` 折叠后，尾部的 `,` 不是必需的。而处在单行中，则必需加逗号。

## range 迭代 string 得到的值

range 得到的索引是字符值（Unicode point / rune）第一个字节的位置，与其他编程语言不同，这个索引并不直接是字符在字符串中的位置。

注意一个字符可能占多个 rune，比如法文单词 café 中的 é。操作特殊字符可使用[norm](https://link.juejin.im?target=https%3A%2F%2Fgolang.org%2Fpkg%2Fvendor%2Fgolang_org%2Fx%2Ftext%2Funicode%2Fnorm%2F) 包。

for range 迭代会尝试将 string 翻译为 UTF8 文本，对任何无效的码点都直接使用 0XFFFD rune（�）UNicode 替代字符来表示。如果 string 中有任何非 UTF8 的数据，应将 string 保存为 byte slice 再进行操作。

```go
func main() {
	data := "A\xfe\x02\xff\x04"
	for _, v := range data {
		fmt.Printf("%#x ", v)	// 0x41 0xfffd 0x2 0xfffd 0x4	// 错误
	}

	for _, v := range []byte(data) {
		fmt.Printf("%#x ", v)	// 0x41 0xfe 0x2 0xff 0x4	// 正确
	}
}
```

## 按位取反

很多编程语言使用 `~` 作为一元按位取反（NOT）操作符，Go 重用 `^` XOR 操作符来按位取反。同时 `^` 也是按位异或（XOR）操作符。

一个操作符能重用两次，是因为一元的 NOT 操作 `NOT 0x02`，与二元的 XOR 操作 `0x22 XOR 0xff` 是一致的。

Go 也有特殊的操作符 AND NOT `&^` 操作符，不同位才取1。

##　跳出 for-switch 和 for-select 代码块

没有指定标签的 break 只会跳出 switch/select 语句，若不能使用 return 语句跳出的话，可为 break 跳出标签指定的代码块：

```go
// break 配合 label 跳出指定代码块
func main() {
loop:
	for {
		switch {
		case true:
			fmt.Println("breaking out...")
			//break	// 死循环，一直打印 breaking out...
			break loop
		}
	}
	fmt.Println("out...")
}
复制代码
```

`goto` 虽然也能跳转到指定位置，但依旧会再次进入 for-switch，死循环。





# 参考

>   https://juejin.im/post/5cad92e8f265da036d79a11f