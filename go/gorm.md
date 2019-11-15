# gorm

>    [参考笔记](https://www.cnblogs.com/shijingjing07/p/10315411.html)

## Problems

### 表名一致问题

定义的表结构名，必须要与数据库中的**表名**一致。所以数据库中表名为`table`，那么在golang代码中结构必须定义为`Table`，而不能是`TableModel`

除此之外，默认的是表名是**复数形式**，即定义的结构体为`UserTest`，gorm查找的表名为`user_tests`。

默认的复数形式可以取消

```go
// 全局禁用表名复数
db.SingularTable(true)
```

但是有一个方法可以完全避免这个问题

```go
func (evaluation *CourseEvaluationModel) TableName() string {
	return "course_evaluation"
}
```

如上，定义的结构体名为`CourseEvaluationModel`，只要声明一个方法`TableName`，返回数据库的表名`course_evaluation`即可。