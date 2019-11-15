# Problems

## http请求response返回的数据乱码问题

+   [Golang 中HTTP请求头 Accept-Encoding 注意事项](https://emacsist.github.io/2018/04/04/golang-中http请求头-accept-encoding-注意事项/)

    链接：response返回的数据乱码问题

## 闭包

闭包函数接收的参数不要是地址

```go
for _, evaluation := range *evaluations {
	wg.Add(1)
	go func(evaluation *model.CourseEvaluation) {
		defer wg.Done()
        fmt.Println(evaluation)
		
	}(&evaluation)
```

如上，goroutine中接收的evaluation的值是一样的

应当将传址改为传值：


```go
for _, evaluation := range *evaluations {
	wg.Add(1)
	go func(evaluation model.CourseEvaluation) {
		defer wg.Done()
        fmt.Println(evaluation)
		
	}(evaluation)
```