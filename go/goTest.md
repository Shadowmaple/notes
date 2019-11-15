# 测试

go test

编译的目标：×_test.go文件

测试函数：

+   功能测试函数：以Test为前缀，检测程序逻辑的正确性
+   基准测试函数：以Benchmark为前缀，测试程序的性能
+   示例函数：以Example为前缀，提供机器检查过的文档



功能测试函数

Test函数：

```go
func TestName(t *testing.T) {
	// ...
}
```

参数t提供了汇报测试失败和日志记录的功能。

```shell
t.Log t.Logf # 正常信息
t.Error t.Errorf # 测试失败信息
t.Fatal t.Fatalf # 致命错误，测试程序退出的信息
t.Fail # 当前测试标记为失败
t.Failed # 查看失败标记
t.FailNow # 标记失败，并终止当前测试函数的执行，需要注意的是，我们只能在运行测试函数的 Goroutine
t.Skip # 调用 t.Skip 方法相当于先后对 t.Log 和 t.SkipNow 方法进行调用，而调用 t.Skipf 方
t.Parallel # 标记为可并行运算
```

要查看更详细的执行信息可以执行 `go test -v `：

```shell
# 输出包中每个测试用力的名称和执行的时间
$ go test -v
```

如果要执行测试 N 次可以使用 -count N ：

```shell
$ go test -v -count 2
```



黑盒测试

白盒测试