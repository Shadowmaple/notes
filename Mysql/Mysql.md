# Mysql

# 了解数据库
数据库(database)是一个以某种有组织的方式存储的数据集合，是保存有组织的数据的容器，通常是一个文件或一组文件

表(table)是某种特定类型数据的结构化清单

模式(schema)是关于数据库和表的布局以及特性的信息

列(column)是表中的一个字段，存储着表中某部分的信息。所有表都是由一个或多个列组成的

行(row)是表中的一个记录

主键(primary key) 是唯一标识表中每行的一列或一组列，用来表示一个特定的行

字句(clause) 用于构成sql语句

## SQL
SQL ：结构化查询语言(Structured Query Language)，是一种专门用来与数据库通信的语言

## MySQL
是一种数据库软件(DBMS)

# 安装

>   [官网下载](https://dev.mysql.com/downloads/)
>
>   [官网apt安装步骤](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/)
>
>   [安装笔记](https://github.com/Shadowmaple/notes/blob/master/Mysql/install.md)

## 启动&停止服务
```shell
shell> sudo service mysql status
shell> sudo service mysql stop
shell> sudo service mysql start
```

## 连接
进入mysql命令行模式
```shell
$ sudo mysql
# 或
$ mysql -uroot -p
```

## 导入样例数据

>   https://blog.csdn.net/duhena0384/article/details/80396542



# 基本操作

ps：关键字都是大写，但因为大小写不敏感，所以可以用小写代替

结束： ; 或 \g

退出： quit 或 exit

帮助： help 或 \h

**注意：结尾要加分号( ; )！**

```mysql
show databases;
use <db_name>;
show tables;
show columns from <table_name>;

select * from <table_name>;
select <column_name> from <table_name>;
```
```mysql
help show;  #显示可用的show命令
```

## 创建数据库
```mysql
Syntax:
CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name
    [create_specification] ...
```
例子

```mysql
mysql> create database mytest;
Query OK, 1 row affected (0.01 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| mytest             |
| performance_schema |
| sample             |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```
## 创建表
```mysql
Syntax:
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    (create_definition,...)
    [table_options]
    [partition_options]
```
# 添加用户

## 添加用户

```mysql
# 无密码登录
create user 'lawler'@'localhost' identified by "";
# 密码登录
create user 'lawler'@'localhost' identified by "12345";
```

## 赋予权限

```mysql
# 所有权限
grant all privileges on *.* to 'lawler'@'localhost';
# 某项数据库所有权限
GRANT privileges ON databasename.* TO 'username'@'host'
# 数据库的某个表的查找和插入权限
GRANT SELECT, INSERT ON databasename.tablename TO 'username'@'host'
```

PS: 

privileges - 用户的操作权限，如SELECT , INSERT , UPDATE 等(详细列表见该文最后面)。如果要授予所的权限则使用`ALL`；databasename - 数据库名，tablename-表名，如果要授予该用户对所有数据库和表的相应操作权限则可用*表示，如*.`*.`

## 查看权限

```mysql
show grants for 'lawler'@'localhost'; 
```

## 删除用户

```mysql
drop user  'lawler'@'localhost'; 
```



# 检索数据

## 检索列
```mysql
# 所有列
select * from <table_name>;
# 单个列
select <column_name> from <table_name>;
# 多个列
select <column1>, <column2>, <column3> from <table_name>;
```
返回的数据是乱序的

## 检索不同行
使用 DISINCT 关键字，返回无重复的该列数据，即对该列数据进行去重操作
```mysql
mysql> SELECT DISTINCT <column> FROM <table>;

mysql> select distinct vend_id from products;
```
## 限制结果
使用 LIMIT 子句
```mysql
# 返回不多于6行
mysql> select prod_name from products limit 6;

# 指定要检索的开始行和行数，从行6开始的3行
mysql> select prod_name from products limit 6, 3;

# 上同，即为从行6取3行
mysql> select prod_name from products limit 3 offset 6;
```
**注意：从 0 行开始！第一行为行0，因此 LIMIT 1,1 将检索出第二行而不是第一行**

## 完全限定的列明和表名
```mysql
mysql> select products.prod_name from sample.products;
```

## 排序检索数据
使用 ORDER BY 子句。ORDER BY 子句取一个或多个列的名字，据此对输出进行排序。
```mysql
# 子句使用检索的列，对该列以字母顺序排序，默认升序
mysql> select prod_name from products order by prod_name;
# 子句可用非检索的列排序
mysql> select prod_name from products order by prod_price;

# 降序，使用 DESC 关键字
mysql> select prod_name from products order by prod_name DESC;

# 按多个列排序，首先价格排列，对相同的再进行字母排列
mysql> select prod_id, prod_id, prod_name from products
    ->  order by prod_price, prod_name;
```
DESC 关键字只应用到直接位于其前面的列名，对多个列进行降序，则需对每个列指定DESC关键字

ASC 关键字为升序，默认

# 过滤数据 —— WHERE语句
使用 WHERE 子句，不区分大小写
## 条件操作符过滤
```mysql
# 相等匹配
mysql> select prod_name, prod_price from products 
    ->  where prod_price = 2.5;

# between, where 和 order by
mysql> select prod_name, prod_price from products 
    ->  where prod_price between 5 and 10
    ->  order by prod_price;

# 若含有空值的列，则返回，否则不返回数据
mysql> select cust_id
    -> from customers
    -> where cust_email IS NULL;
```

操作符：用来联结或改变WHERE子句中的子句的关键字

| 条件操作符 | 说明|
| :---: | :---: |
| = | 等于 |
| <> | 不等于 |
| != |不等于 |
| BETWEEN | 两者之间 |
| <,<=,>,>= | ...(不必多说) |

注意：

1. 同时使用 WHERE 和 ORDER BY 时，WHERE 应在前，否则会报错
2. 使用 BETWEEN 时注意左低位，右高位，否则返回的是无数据

## 逻辑操作符过滤
```mysql
# OR操作符
mysql> select prod_name, prod_price from products
    -> where prod_price in (10, 8.99);

# IN操作符
mysql> select prod_name, prod_price from products
    -> where prod_price in (10, 8.99);
```
|逻辑操作符| 说明 |
| :---: | :---: |
| IN | 指定范围，范围中每个条件都可以匹配；在圆括号中用逗号分隔 |
| AND|
| OR |
| NOT |

注意：

IN 不等于 BETWEEN ，而是相当于多个 OR！`x IN (a, b)` 相当于 `x = a OR x = b`

## 通配符过滤
通配符：用来匹配一部分的特殊字符

搜索模式：由字面值、通配符或两者组合构成的搜索条件

| 通配符 | 说明 |
| :---: | :---: |
| % | 匹配任意个字符，包括0个字符 |
| _ | 只匹配单个字符，不包括0个字符 |

LIKE 操作符：

为在搜索子句中使用通配符，必须使用**LIKE**操作符。LIKE指示MYSQL，后跟的搜索模式利用通配符匹配而不是直接相等匹配进行比较。

```mysql
# (%)匹配符
mysql> select prod_id, prod_name from products
    -> where prod_name like 'jet%';
    
# (_)匹配符
mysql> select prod_id, prod_name from products
    -> where prod_name like '_ ton anvil';
```

注意：
1. 匹配默认不区分大小写
2. 注意尾空格的存在
3. % 不能匹配 NULL
4. 使用通配符的搜索效率较低，因此不要过度使用通配符
5. 尽量不要将其用在搜索模式的开始处，将通配符置于开始处，搜索效率是最低的
6. 注意通配符的放置位置

## 正则表达式过滤
使用 REGEXP 关键字

LIKE 与 REGEXP 的区别：

LIKE 匹配整个列， REGEXP则在列值内进行匹配。

```mysql
mysql> select prod_id, prod_name from products
    -> where prod_name regexp '[123] Ton';
```

在 Mysql 中，转义特殊字符需要两个反斜杠`（\\）`。因为一个由 Mysql 解释，另一个由正则表达式库解释。
( ^ )有两种用法。在集合中，用它来否定该集合（`'[^123]'`），在集合外，表示串的开始处（`'^[123]'`）。

正则表达式的特性，只要匹配到相应的字段，那么这个列的值就匹配。如下，简单的正则表达式测试：

```mysql
mysql> SELECT 'hello' REGEXP '[0-9]';
+----------------------------+
| 'hello' REGEXP '[0-9]' |
+----------------------------+
|                      0 					 |
+----------------------------+
1 row in set (0.00 sec)
# 返回0表示未匹配

mysql> SELECT 'hello4' REGEXP '[0-9]';
+------------------------------+
| 'hello4' REGEXP '[0-9]' |
+------------------------------+
|                       1 						|
+------------------------------+
1 row in set (0.00 sec)
# 返回1表示匹配
```

> 更多姿势详见正则表达式规则
>
> [学习文档](https://github.com/ziishaned/learn-regex/blob/master/translations/README-cn.md)


# 计算字段
## 拼接字段
字段(field)：基本与列的意思相同，经常互换使用，但字段通常用在计算字段的连接上

拼接(concatenate)：将值联结到一起构成单个值

使用 `Concat()` 函数来拼接两个列。

`Concat()` 拼接串，即把多个串连接起来形成一个较长的串。其需要一个或多个指定的串，各个串之间的用逗号分隔。

```mysql
mysql> select concat(vend_name, ' (', vend_country, ')')
    -> from vendors order by vend_name;
+--------------------------------------------+
| concat(vend_name, ' (', vend_country, ')') |
+--------------------------------------------+
| ACME (USA)                                 |
| Anvils R Us (USA)                          |
| Furball Inc. (USA)                         |
| Jet Set (England)                          |
| Jouets Et Ours (France)                    |
| LT Supplies (USA)                          |
+--------------------------------------------+
6 rows in set (0.00 sec)
```

赋予别名—— AS 关键字
```mysql
mysql> select concat(vend_name, ' (', vend_country, ')')
    -> as vend_title
    -> from vendors order by vend_name;
+-------------------------+
| vend_title              |
+-------------------------+
| ACME (USA)              |
| Anvils R Us (USA)       |
| Furball Inc. (USA)      |
| Jet Set (England)       |
| Jouets Et Ours (France) |
| LT Supplies (USA)       |
+-------------------------+
6 rows in set (0.00 sec)
```

## 执行算数计算
算术操作符：+,  -,  *,  /

测试计算

```mysql
mysql> select 6/2;
+--------+
| 6/2    |
+--------+
| 3.0000 |
+--------+
1 row in set (0.00 sec)
```

实际应用

```mysql
mysql> select quantity*item_price as expanded_price
    -> from orderitems where order_num = 20005;
+----------------+
| expanded_price |
+----------------+
|          59.90 |
|          29.97 |
|          50.00 |
|          10.00 |
+----------------+
4 rows in set (0.00 sec)
# 输出的 expanded_price 列为一个计算字段
```
# 数据处理函数
函数的可移植性不强

## 文本处理函数
| 函数 | 说明 |
| :---: | :---: |
| Left() | 返回串左边的字符 |
| Length() | 返回串的长度 |
| Locate() | 找出串的一个子串|
| Lower() | 转换为小写|
| LTrim() | 去掉左边的空格|
| RTrim() | 去掉右边的空格|
| Right() | 返回串右边的字符|
| Soundex() | 返回串的SOUNDEX值|
| SubString() | 返回子串的字符|
| Upper() | 转换为大写 |

Soundex() 是一个将任何文本串替换为描述其语音表示的字母数字模式的算法，即根据发音确定是否符合

## 日期和时间处理函数
| 函数 | 说明 |
| :---: | :---: |
| AddDate() | 增加一个日期 |
| AddTime() | 增加一个时间 |
| CurDate() |  返回当前日期 |
| CurTime() | 返回当前时间 |
| Date() | 返回日期时间的日期部分|
| DateDiff() | 计算两个日期之差|
| Date_Add() | 高度灵活的日期运算函数|
| Date_Format() |  返回一个格式化的日期或时间串|
| Day() | 返回一个日期的天数部分|
| DayOfWeek() | 返回一个对应日期的星期几|
| Hour() | 返回小时部分|
| Minute() | 返回分钟部分|
| Month() | 返回月份|
| Now() | 返回当前日期和时间|
| Second() | 返回秒部分|
| Time() | 返回一个日期时间的时间部分|
| Year() | 返回年份|

```mysql
# 返回当前日期和时间
mysql> select now();
+---------------------------+
| now()              		 		 |
+---------------------------+
| 2019-04-29 23:12:33 |
+---------------------------+
1 row in set (0.00 sec)
```

## 数值处理函数
| 函数 | 说明 |
| :---: | :---: |
| Abs() | 绝对值|
| Cos() | 余弦|
| Exp() | 返回指数值|
| Mod() | 余数|
| Pi() | 圆周率|
| Rand() | 返回一个随机数|
| Sin() | 正弦|
| Sqrt() | 平方根|
| Tan() | 正切|

# 汇总数据
## 聚集函数
聚集函数：运行在行组上，计算和返回单个值的函数

| 聚集函数 | 说明 |
| :---: | :---: |
| AVG() | 返回某列的平均值 |
| COUNT() | 返回某列的行数 |
| MAX() | 返回某列的最大值 |
| MIN() | 返回最小值 |
| SUM() | 返回某列值之和 |

例：

```mysql
mysql> select avg(prod_price) as avg_price from products;
```

对于列值为 NULL 的情况：

一般都忽略，但是 `COUNT()` 函数有点区别。`COUNT(*)` 时不忽略列值为 NULL 的行，而 `COUNT(Column)` 对某一列进行计算时，会忽略 NULL 的行

## 聚集不同值
使用 DISTINCT 参数，默认为 ALL 参数
```mysql
mysql> select avg(distinct prod_price) from products;
```
注意：

若指定列明，则 DISTINCT 只能用于 COUNT(Column)。DISTINCT 不能用于 COUNT(*)，也不允许使用 COUNT(DISTINCT)，否则会产生错误。

# 分组数据
分组允许把数据分为多个逻辑组，以便能对每个组进行聚集计算

使用 `GROUP BY` 语句

使用 `WITH ROLLUP` 关键字，可以得到每个分组以及每个分组汇总级别的值（针对每个分组）
```mysql
mysql> select vend_id, count(*) from products group by vend_id;
+---------+----------+
| vend_id | count(*) |
+---------+----------+
|    1001 |        3 |
|    1002 |        2 |
|    1003 |        7 |
|    1005 |        2 |
+---------+----------+
4 rows in set (0.04 sec)
# 实际上就是分类汇总

# 加上 WITH ROLLUP 关键字
mysql> select vend_id, count(*) from products group by vend_id with rollup;
+---------+----------+
| vend_id | count(*) |
+---------+----------+
|    1001 |        3 |
|    1002 |        2 |
|    1003 |        7 |
|    1005 |        2 |
|    NULL |       14 |
+---------+----------+
5 rows in set (0.00 sec)
```
注意：
1. GROUP BY 子句可以包含任意数目的列。这使得能对分组进行嵌套，为数据分组提供更细致的控制。
2. 若嵌套了分组，数据将在最后规定的分组上进行汇总。即在建立分组时，指定的所有列都一起计算，所以不能从个别的列取回数据。
3. GROUP BY子句中列出的每个列都必须是检索列或有效的表达式，但不能是聚集函数。若在SELECT中时候用表达式，则必须在GROUP BY子句中指定相同的表达式，不能使用别名。
4. 除聚集计算语句外，SELECT 语句中的每个列都必须在GROUP BY子句中给出
5. 若分组列中具有 NULL 值，则NULL将作为一个分组返回；若有多行NULL值，则将它们分为一组。
6. GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前。

## 过滤分组
使用 HAVING 关键字进行过滤分组。

由于 WHERE 过滤指定的是行而不是分组，所以不能完成过滤分组的任务。

HACING 支持所有 WHERE 的操作符，即句法相同，只是关键字有差别。

两者差别：

WHERE在数据分组前进行过滤，而HAVING在数据分组后进行过滤。WHERE排除的行不包括在分组中，这可能会改变计算值，从而影响 HAVING 子句中基于这些值过滤掉的分组。

```mysql
mysql> select cust_id, count(*) as orders from orders 
    -> group by cust_id
    -> having count(*) >= 2;
```

## 分组和排序
GROUP BY 输出的可能不是分组的顺序，所以不一定是有序的，因此要想保证有序，就要使用 ORDER BY 子句。

注意子句的顺序：
```mysql
SELECT -> FROM -> WHERE -> GROUP BY -> HAVING -> ORDER BY -> LIMIT
```
# 子查询
子查询， 即嵌套在其他查询中的查询

相关子查询：涉及外部查询的子查询

```mysql
# 在WHERE子句中使用子查询，利用子查询实现过滤
mysql> select cust_id from orders
    -> where order_num in (select order_num
    -> from orderitems
    -> where prod_id = 'TNT2');

# 作为计算字段使用子查询
mysql> select cust_name, cust_state, 
    -> (select count(*) from orders
    -> where orders.cust_id = customers.cust_id) as orders
    -> from customers
    -> order by cust_name;
```
注意：

列必须匹配。在WHERE子句中使用子查询，应保证 SELECT 语句具有与 WHERE 子句中相同数目的列。子查询可以单列与单列匹配，也可以多列与多列匹配。

# 联结表
外键(foreign key)：某个表中的一列，包含另一个表的主键值，定义了两个表的关系

可伸缩性(scale)：能够适应不断增加的工作量而不失败

联结：联结是一种机制，用来在一条SELECT语句中关联表

内部联结：也称等值联结，是基于两个表之间的相等测试

笛卡尔积：由没有联结条件的表关系返回的结果。检索处的行的数目将是第一个表中的行数与第二个表行数的乘积


```mysql
# 创建联结，规定要联结的所有表以及他们如何关联即可
mysql> select vend_name, prod_name, prod_price
    -> from vendors, products
    -> where vendors.vend_id = products.vend_id
    -> order by vend_name, prod_name;

# 明确内部联结的类型，语法稍有不同，但效果与上面的相同
mysql> select vend_name, prod_name, prod_price
    -> from vendors inner join products
    -> on vendors.vend_id = products.vend_id
    -> order by vend_name, prod_name;
    
# 联结多个表
mysql> select prod_name, vend_name, prod_price, quantity
    -> from orderitems, products, vendors
    -> where products.vend_id = vendors.vend_id
    -> and orderitems.prod_id = products.prod_id
    -> and order_num = 20005;
```
注意：
1. 应使用完全限定列名
2. 应保证所有联结都有 WHERE 子句
3. WHERE 子句定义联结条件比较简单；`INNER JOIN... ON...`语句能明确使用的联结条件，但有时候会影响性能
4. 要注意考虑性能。不要联结不必要的表，联结的表越多，性能下降越厉害。
5. 表别名只在查询执行中使用，与列别名不一样，表别名不返回到客户机。

