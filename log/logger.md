## log 实践

#### 作用

+   排查 bug，快速定位
+   性能瓶颈
+   数据分析

#### 级别

+   debug
+   info
+   warning
+   error
+   fatal

#### 参数

+   记录时间
+   记录位置（文件、行号）
+   请求-响应时间
+   关键信息
+   ……

#### 打 log

```go
func ginHandler(c *gin.Context) {
	fmt.Println("hello, world.")
    if err := Foo(); err != nil {
    	log.Error("Foo function error", err)
        // ...
    }
}

func Foo() error {
    if err := UpdateUser(); err != nil {
    	log.Error("UpdateUser error", err)
        return err
    }
}

func UpdateUser() error {
}
```



#### 日志监控



#### Golang 日志包

+   zap