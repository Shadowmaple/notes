# Mongo的Go驱动使用

# 认识

目前网上有两个go的mongo驱动库相对来说比较流行，一个是社区的 mgo，另一个是官方推出的"Mongo-go-driver"。mgo 是14年推出的，已经流行很久了，网上教程也非常的多，基本上google搜“mongo的go使用”的都是mgo，但是mgo已经一年多没更新过了（算是废弃了），基本上和官方的mongo脱节，不便于使用；而官方的“Mongo-go-driver”虽然推出还不到一年，网上教程也非常少，官方也没有推出很好的教程，但是长久来看，它必定是主流。所以就算现在使用官方的驱动较为困难，但也不应该成为放弃的理由。就此做下简单的学习笔记。

# 基本操作

## 依赖包

```go
"context"
    
"go.mongodb.org/mongo-driver/bson"
"go.mongodb.org/mongo-driver/mongo"
"go.mongodb.org/mongo-driver/mongo/options"
```

这四个基本上是必须的，其它的类似 fmt, log 就另说了



## 连接

```go
mongodbUrl := "mongodb://127.0.0.1:27017"
clientOptions := options.Client().ApplyURI(mongodbUrl)

client, err := mongo.Connect(context.TODO(), clientOptions)
if err != nil {
	fmt.Println("Failed")
    log.Fatal(err)
}

// 连接集合
collection := client.Database("table").Collection("test")
```



## 定义映射

```go
type User struct {
	Name 	string	`bson:"name"`
	Sex		string	`bson:"sex"`
	Age 	int 	`bson:"age"`
}
```



> Before we start sending queries to the database, it's important to understand how the Go Driver works with BSON objects. JSON documents in MongoDB are stored in a binary representation called BSON (Binary-encoded JSON). Unlike other databases that store JSON data as simple strings and numbers, the BSON encoding extends the JSON representation to include additional types such as int, long, date, floating point, and decimal128. This makes it much easier for applications to reliably process, sort, and compare data.



## 添加文档

```go
// 添加单个
if insertId, err := collection.InsertOne(context.TODO(), User{Name: "Nick"}); err != nil {
	fmt.Println(err)
} else {
	fmt.Println(insertId)
    // 输出结果：&{ObjectID("5d5391f7b9023b91ee179295")}
}

// 添加多个
users := []interface{}{
	User{Name:"Mark", Age:18, Sex:"male"},
	User{Name:"Ark", Age:12},
	User{Name:"Bob", Sex:"male"},
}
if insertId, err := collection.InsertMany(context.TODO(), users); err != nil {
	fmt.Println(err)
} else {
	fmt.Println(insertId)
	// 输出结果：&{[ObjectID("5d5391f7b9023b91ee179296") ObjectID("5d5391f7b9023b91ee179297") ObjectID("5d5391f7b9023b91ee179298")]}
}
```

`InsertOne`和`InsertMany`返回的第一个值都是有关插入文档的 Id，Id是ObjectId类型，在Mongodb中存储的字段名是"_id"，这是自动生成的，也可以自行在结构体中定义这个字段。

## 查询

```go
user := User{}
if err := collection.FindOne(context.TODO(), bson.M{"age": 18}).Decode(&user); err != nil {
	fmt.Println(err)
} else {
	fmt.Println(user)	
    // 输出结果： {Mark male 18}
}

// 获得多个结果
cur, err := collection.Find(context.TODO(), bson.M{"sex": "male"})
if err != nil {
	fmt.Println(err)
}
// 必须要放在错误判断之后
defer cur.Close(context.TODO())

// 迭代获得每一个查询结果
for cur.Next(context.TODO()) {
	var user User
	if err := cur.Decode(&user); err != nil {
		fmt.Println(err)
	}
	fmt.Println(user.Name, user.Age)
}
```

查询的结果理所当然的是一条或一组记录了



## 修改

```go
// 更新
if modifyResult, err := collection.UpdateOne(
	context.TODO(),
	bson.M{"name": "Mark"},
	bson.M{"$set": User{Name:"Lily", Sex:"female"}}); err != nil {
		fmt.Println(err)
} else {
	fmt.Println(modifyResult)	
	// 输出结果： &{1 1 0 <nil>}
}

// 替换
if modifyResult, err := collection.ReplaceOne(
	context.TODO(),
	bson.M{"name": "Nick"},
	User{Name: "Jack", Age: 25, Sex: "female"}); err != nil {
		fmt.Println(err)
} else {
	fmt.Println(modifyResult)
    //输出结果： &{1 1 0 <nil>}
}
```

更新和替换的区别很好理解，更新是对原有文档的某些字段进行修改，而替换则是对整个文档的取代。

`UpdateOne`和`UpdateMany`的更新值必须要有`$`，作为更新的行为，以上的例子是`set`设置，还有递增一的`iner`



`UpdateOne`和`ReplaceOne`返回的结果都是`UpdateResult`的指针。`UpdateResult`有四个值，分别是匹配到的文档数、修改的文档数、插入的文档数和插入的 ObjectId。这样就可以理解为什么输出的结果是 `&{1 1 0 <nil>}` 了。

```go
type UpdateResult struct {
    // The number of documents that matched the filter.
    MatchedCount int64
    // The number of documents that were modified.
    ModifiedCount int64
    // The number of documents that were upserted.
    UpsertedCount int64
    // The identifier of the inserted document if an upsert took place.
    UpsertedID interface{}
}
```



## 删除

```go
// 删除单个
if deleteCount, err := collection.DeleteOne(context.TODO(), bson.M{"age": "Nick"}); err != nil {
	fmt.Println(err)
} else {
	fmt.Println(deleteCount)	
	// 输出结果： &{0}
}

// 删除多个
if deleteCount, err := collection.DeleteMany(context.TODO(), bson.M{"sex": "male"}); err != nil {
	fmt.Println(err)
} else {
	fmt.Println(deleteCount)	
	// 输出结果： &{2}
}
```

删除的方法返回的第一个值是关于删除的记录（文档）数。



# 高级应用

## 账户加密连接Mongodb

在实际的开发当中，我们需要以一个有确定权限的用户身份对数据库进行操作，而不是在本地默认无用户式地连接。

关于如何添加用户，这是有关Mongodb的操作，在这里就不赘述了。

具体的格式如下：

```go
mongodb://<user>:<password>@<mongodb_url>[/<mongo_db>]
```

以mongodb开头，接着账户名、密码、所连接的mongo的地址，之后可以接一个连接的mongo的数据库名。因为只有在admin中添加的账户才拥有访问全部数据库的权限，所以如果不是在admin的数据库中添加的用户名却没加上其所在的db，那么连接就会报错，显示身份验证不通过。

比如连接本地mongodb（默认本地地址和端口），且这是个在table中添加的“table.admin”账户：

```go
mongodbUrl := "mongodb://admin:secret@127.0.0.1:27017/table"
```



## 设置上下文环境

将`context.TODO()`作为mongo操作的上下文是十分原始的，一般情况下无伤大雅，但实际上很多时候会涉及到上下文的操作，例如用户的身份验证。在用 gin 框架开发的时候会用到 gin 上下文 `gin.context`。但如果只是mongodb的程序设置一个良好的上下文环境也是很有必要的。下面就设置了一个若超出规定时间内停止相关程序服务、释放资源上下文环境。

```go
client, err := mongo.NewClient(options.Client().ApplyURI("mongodb://localhost:27017"))
if err != nil {
	log.Fatal(err)
}

ctx, _ := context.WithTimeout(context.Background(), 10*time.Second)
if err := client.Connect(ctx); err != nil {
	log.Fatal(err)
}
```



## `bson.M` 和`bson.D`

本人在学习的过程中，对这两者常常产生困惑，因为在一些地方出现 bson.M，另一些地方用 bson.D。但官方的演示代码中，`bson.D`总是出现，而`bson.M`却出现的很少。两者其实很类似，所谓区别其实就是一个是否有序的问题。

bson.M 是一个映射类型，而 bson.D 也是一个映射，但它却是有序的。

什么意思？看个例子：假如之前的文档没有删除完，还留有 User  结构体（字段分别为name, sex, age）的记录，那么再添加如下两条记录

```go
// 测试用，直接连err都不要了，实际的开发中显然不能如此草率
_, _ = collection.InsertOne(
	context.TODO(),
	bson.M{"city": "Wuhan", "name": "Ark", "Language": "Java", "age": 18})

_, _ = collection.InsertOne(
	context.TODO(),
	bson.D{
		{"city", "Hangzhou"},
		{"name", "Nick"},
		{"Language", "Go"},
		{"age", 20},
	})


// 在mongodb中的存储结果分别为
{ "_id" : ObjectId("5d53c874a22991fa217d4549"), "age" : 18, "city" : "Wuhan", "name" : "Ark", "Language" : "Java" }
{ "_id" : ObjectId("5d53c874a22991fa217d454a"), "city" : "Hangzhou", "name" : "Nick", "Language" : "Go", "age" : 20 }
```

显然，bson.D 的文档还保有定义的字段顺序，而 bson.M 却是乱了。这就是两者的区别，如此来看，还是用bson.D 比较保险，所以一般来说作为过滤的接口(filter)用 bson.M ，而存值用 bson.D 比较好。



除此之外，bson的D家族类型还有 bson.A，bson.E。bson.A 是一个bson的数组，bson.E 是一个在D中的单个元素（基本用不到）。

```go
bson.A{"a", "b", "c", 12}
```



## ObjectId的设置与获取

在实际的开发中，有时候我们需要获取mongo的“_id“字段。这就需要我们事先定义这个字段了，而不是由系统自动生成。

```go
import (
	"go.mongodb.org/mongo-driver/bson/primitive"
    
    // ...
    // reset package
)

type User struct {
	Id 		primitive.ObjectID 	`bson:"_id"`
	// ...
}
```

这样就可以像获取其它字段一样得到id了，但这时得到的id是objectId类型的，我们还要将他转换为string类型

```go
// Hex方法将objectid转为string
idStr := id.Hex()
```

还可以在添加文档的时候选择手动生成id

```go
// 手动生成objectId
id := primitive.NewObjectID()
```



