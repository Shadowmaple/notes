# gin

## Problems

### binding required要求数据不为空值

结构体binding 使用required时，传入的参数不允许是空值。这个“空值”的概念是对于go结构体中定义的字段类型而言的，即`int`、`uint`不允许接收0，string不允许接收`""`

```go
type GetRequest struct {
    Content string `json:"content" binding:"required"`
    Number  uint8  `json:"number" binding:"required"`
}
```

如上，传入的json数据，content不能是`""`，number不能是0

>   死坑的