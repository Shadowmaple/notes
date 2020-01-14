# Mysql

# 了解数据库
数据库(database)是一个以某种有组织的方式存储的数据集合，是保存有组织的数据的容器，通常是一个文件或一组文件

表(table)是某种特定类型数据的结构化清单

模式(schema)是关于数据库和表的布局以及特性的信息

列(column)是表中的一个字段，存储着表中某部分的信息。所有表都是由一个或多个列组成的

行(row)是表中的一个记录

主键(primary key) 是唯一标识表中每行的一列或一组列，用来表示一个特定的行

字句(clause) 用于构成sql语句

SQL ：结构化查询语言(Structured Query Language)，是一种专门用来与数据库通信的语言

MySQL：是一种数据库软件(DBMS)

# 安装

>   [官网下载](https://dev.mysql.com/downloads/)
>
>   [安装步骤](https://github.com/Shadowmaple/notes/blob/master/Mysql/install.md)

# 基本操作

## 启动&停止服务

```shell
shell> sudo service mysql status
shell> sudo service mysql stop
shell> sudo service mysql start
```

## 连接
进入mysql命令行模式
```shell
$ mysql -h 127.0.0.1 -P 3306 -u root -p123
# 直接进入test数据库
$ mysql -h 127.0.0.1 -P 3306 -u root -p123 test
```

ps：关键字都是大写，但因为大小写不敏感，所以可以用小写代替

结束： `;` 或` \g`

退出： `quit`或 `exit`

帮助： `help` 或`\h`

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
CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] db_name [create_specification] ...
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
    
CREATE TABLE <表名>
(
    <列名1> <数据类型> <该列所需约束> ,
	<列名2> <数据类型> <该列所需约束> ,
	<列名3> <数据类型> <该列所需约束>
);  
```

数据类型有 INTEGER, CHAR, DATE 等，约束条件有 NOT NULL, PRIMARY KEY (主键)，DEFAULT 0  等。

例如：

```mysql
create table Addressbook(
    regist_no	 integer 	  not null,
    name		 varchar(128) not null,
    address		 varchar(256) not null,
    tel_no 		 char(10),
    mail_address char(20), 
    primary key (regist_no)
);
```

## 删除表

```mysql
DROP TABLE  <表名>;
```

## 查看表的定义

### 查询结果显示

```mysql
desc <表名>;
describe <表名>;
show columns from <表名>;

-- 结果：
+------------+------------------+------+-----+---------+----------------+
| Field      | Type             | Null | Key | Default | Extra          |
+------------+------------------+------+-----+---------+----------------+
| id         | int(10) unsigned | NO   | PRI | <null>  | auto_increment |
| sid        | varchar(10)      | NO   |     | <null>  |                |
| username   | varchar(25)      | YES  |     | <null>  |                |
| avatar     | varchar(255)     | YES  |     | <null>  |                |
| is_blocked | tinyint(4)       | NO   |     | 0       |                |
+------------+------------------+------+-----+---------+----------------+
```

### SQL语句显示

```mysql
show create table <表名>;

-- 结果：
+-------+--------------------------------------------------------+
| Table | Create Table                                           |
+-------+--------------------------------------------------------+
| user  | CREATE TABLE `user` (                                  |
|       |   `id` int(10) unsigned NOT NULL AUTO_INCREMENT,       |
|       |   `sid` varchar(10) NOT NULL COMMENT '学生学号',        |
|       |   `username` varchar(25) DEFAULT NULL,                 |
|       |   `avatar` varchar(255) DEFAULT NULL,                  |
|       |   `is_blocked` tinyint(4) NOT NULL DEFAULT '0',        |
|       |   PRIMARY KEY (`id`)                                   |
|       | ) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
```

## 更新表的定义

在表创建完成之后才发现字段多了或者少了，这时候无需删表重新创建，只要对表更新定义即可。

```mysql
-- 添加字段
ALTER TABLE < 表名 > ADD COLUMN < 列的定义 > ;

-- 删除字段
ALTER TABLE < 表名 > DROP COLUMN < 列名 > ;

-- 字段重命名
ALTER TABLE <表名> CHANGE COLUMN <列名> <新列名> <类型>;

-- 更改字段类型
ALTER TABLE <表名> MODIFY COLUMN <列名> <类型>;
```

如：

```mysql
-- 添加字段
alter table Addressbook add column (postal_code varchar(8) not null);

-- 字段重命名
alter table users change column name nickname varchar(8);

-- 更改字段类型
alter table user modify column name varchar(10) not null;
```

## 更改表名

```mysql
RENAME TABLE <原始表名> to <新表名>;
```

## 注释

1.  单行注释： `--`。要注意后面要空一格，否则不会认为是注释
2.  多行注释：`/*` 和 `*/`。

## 命令结束符号

在书写完一个命令之后需要以下边这几个符号之一结尾：

+   `;`
+   `\g`：与`;`效果等同
+   `\G`：与前两个有所不同，结果会以列垂直的形式展示，在列比较多的情况下较为方便

## 导入样例数据

>   https://blog.csdn.net/duhena0384/article/details/80396542

# 新用户

## 添加用户

```mysql
-- 无密码登录
create user 'lawler'@'localhost' identified by "";
-- 密码登录
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

注意：

privileges - 用户的操作权限，如SELECT , INSERT , UPDATE 等。

如果要授予所的权限则使用`ALL`；如果要授予该用户对所有数据库和表的相应操作权限则可用表示，如 `*.*.`。

## 查看权限

```mysql
show grants for 'lawler'@'localhost'; 
```

## 删除用户

```mysql
drop user 'lawler'@'localhost'; 
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
注意：使用`*`返回的列的顺序是定义时的顺序。

## 检索不同行
使用 DISINCT 关键字，返回无重复的该列数据，即对该列数据进行去重操作
```mysql
mysql> SELECT DISTINCT <column> FROM <table>;

mysql> select distinct vend_id from products;
```
注意：

+   在使用 DISTINCT 时, NULL 也被视为一类数据。 NULL 存在于多行中时 , 也会被合并为一条 NULL 数据。

+   多条列名时，DISTINCT 关键字只能用在第一个列名之前。

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
注：

+   从 0 行开始！第一行为行0，因此 LIMIT 1,1 将检索出第二行而不是第一行

## 完全限定的列明和表名
```mysql
mysql> select products.prod_name from sample.products;
```

## 排序
使用 ORDER BY 子句。ORDER BY 子句取一个或多个列的名字，据此对输出进行排序。ORDER BY 子句中书写的列名称为排序键。

```mysql
# 子句使用检索的列，对该列以字母顺序排序，默认升序
mysql> select prod_name from products order by prod_name;
# 子句可用非检索的列排序
mysql> select prod_name from products order by prod_price;

# 降序，使用 DESC 关键字
mysql> select prod_name from products order by prod_name DESC;

# 多个排序键，首先价格排列，对相同的再进行字母排列
mysql> select prod_id, prod_id, prod_name from products
    -> order by prod_price, prod_name;
```
注：

+   DESC 关键字只应用到直接位于其前面的列名，对多个列进行降序，则需对每个列指定DESC关键字。

+   ASC 关键字为升序，默认升序。

+   由于 ASC 和 DESC 这两个关键字是以列为单位指定的，因此可以同时指定一个列为升序，指定其他列为降序。

+   排序键中包含 NULL 时，会在开头或末尾进行汇总。
+   在 ORDER BY 子句中可以使用 SELECT 子句中定义的别名。
+   在 ORDER BY 子句中可以使用 SELECT 子句中未使用的列和聚合函数。

# 过滤数据 —— WHERE语句
## 条件操作符过滤
```mysql
# 相等匹配
mysql> select prod_name, prod_price from products 
    -> where prod_price = 2.5;

# between, where 和 order by
mysql> select prod_name, prod_price from products 
    -> where prod_price between 5 and 10
    -> order by prod_price;

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

1.  使用 WHERE 子句，不区分大小写
2.  同时使用 WHERE 和 ORDER BY 时，WHERE 应在前，否则会报错
3.  使用 BETWEEN 时注意左低位，右高位，否则返回的是无数据

## 逻辑操作符过滤
```mysql
# OR操作符
mysql> select prod_name, prod_price from products
    -> where prod_price = 10 or prod_price = 8.99;

# IN操作符
mysql> select prod_name, prod_price from products
    -> where prod_price in (10, 8.99);
    
# NOT
mysql> select prod_name, prod_price from products
    -> where not prod_price > 5;
```
|逻辑操作符| 说明 |
| :---: | :---: |
| IN | 指定范围，范围中每个条件都可以匹配；在圆括号中用逗号分隔 |
| AND|
| OR |
| NOT |

注意：

+   IN 不等于 BETWEEN ，而是相当于多个 OR！`x IN (a, b)` 相当于 `x = a OR x = b`。
+   AND 运算符优先于 OR 运算符，可以使用括号。

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

## 字符串处理函数
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
| Concat() | 字符串拼接 |

注：

Soundex() 是一个将任何文本串替换为描述其语音表示的字母数字模式的算法，即根据发音确定是否符合。

Length() 函数是根据字符串的字节数来判断的，即对于中文字符的结果在 utf-8下是一个汉字返回3，而在一些中文字节编码下返回的是2。

Substring函数用法：

```mysql
SUBSTRING (对象字符串 FROM 截取的起始位置 FOR 截取的字符数)
```



## 时间日期处理函数
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
| TO_DAYS(date) | 返回一个天数(从公元0年的天数) |

```mysql
# 返回当前日期和时间
mysql> select now();
+---------------------------+
| now()              		 		 |
+---------------------------+
| 2019-04-29 23:12:33 |
+---------------------------+
1 row in set (0.00 sec)

# 两个日期之差
select DateDiff('2019-02-03', '2019-02-04')
# 返回-1

# 返回天数，用在计算两个日期之差
select to_days('2019-08-27')
# 返回737663
```

## 数值处理函数
| 函数 | 说明 |
| :---: | :---: |
| Abs() | 绝对值|
| Cos() | 余弦|
| Exp() | 返回指数值|
| Mod( 被除数,除数 ) | 求余 |
| Round(对象数值,保留小数的位数) | 四舍五入 |
| Pi() | 圆周率|
| Rand() | 返回一个随机数|
| Sin() | 正弦|
| Sqrt() | 平方根|
| Tan() | 正切|
| Numeric(全体位数,小数位数) | 浮点数数据类型 |

NUMERIC 是大多数 DBMS 都支持的一种数据类型，通过 `NUMBERIC(全体位数 , 小数位数)` 的形式来指定数值的大小，支持小数，应用在创建表的时候。



# 数据汇总
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
# 该列的平均值
mysql> select avg(prod_price) as avg_price from products;
```

注：

+   除了 COUNT() 函数，其它函数都不能将 * 作为参数。
+   MAX / MIN 函数几乎适用于所有数据类型的列。 SUM / AVG 函数只适用于数值类型的列。

+   对于列值为 NULL 的情况：

    一般都忽略，但是 `COUNT()` 函数有点区别。`COUNT(*)` 时不忽略列值为 NULL 的行，而 `COUNT(Column)` 对某一列进行计算时，会忽略 NULL 的行。

+   所有的聚合函数都可以使用 DISTINCT 来去除重复数据。

+   只有 SELECT 子句和 HAVING 子句(以及 ORDER BY 子句)中能够使用聚合函数，WHERE 不能使用聚合函数。

## 聚集不同值
使用 DISTINCT 参数，默认为 ALL 参数
```mysql
mysql> select avg(distinct prod_price) from products;
```
注意：

若指定列明，则 DISTINCT 只能用于 COUNT(Column)。DISTINCT 不能用于 COUNT(*)，也不允许使用 COUNT(DISTINCT)，否则会产生错误。

# 数据分组

## 分组

分组允许把数据分为多个逻辑组，以便能对每个组进行聚集计算。

使用 `GROUP BY` 语句进行分组，且可以通过逗号分隔指定多列。在 GROUP BY 子句中指定的列称为聚合键或者分组列。

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
4. 除聚集计算语句外，不能把聚合键之外的列名书写在 SELECT 子句之中，否则会报错。
5. 若分组列中具有 NULL 值，则NULL将作为一个分组返回；若有多行NULL值，则将它们分为一组。
6. 使用 WHERE 子句进行汇总处理时，会先根据 WHERE 子句指定的条件进行过滤,然后再进行汇总处理。

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

注：

聚合键所对应的条件不应该书写在 HAVING 子句当中，而应该书写在 WHERE 子句当中。

## 分组和排序

GROUP BY 输出的可能不是分组的顺序，所以不一定是有序的，因此要想保证有序，就要使用 ORDER BY 子句。

注意子句的顺序：
```mysql
SELECT -> FROM -> WHERE -> GROUP BY -> HAVING -> ORDER BY -> LIMIT
```
执行顺序：

```mysql
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```



# 子查询

## 子查询

子查询， 即嵌套在其他查询中的查询

相关子查询：涉及外部查询的子查询

```mysql
# 在WHERE子句中使用子查询，利用子查询实现过滤
select cust_id from orders
    where order_num in (
        select order_num
    	from orderitems
    	where prod_id = 'TNT2'
    );

# 作为计算字段使用子查询
select cust_name, cust_state, 
(
    select count(*)
	from orders
	where orders.cust_id = customers.cust_id
) as orders
from customers
order by cust_name;
```
注意：

列必须匹配。在WHERE子句中使用子查询，应保证 SELECT 语句具有与 WHERE 子句中相同数目的列。子查询可以单列与单列匹配，也可以多列与多列匹配。

子查询可以多层嵌套，但是随着子查询嵌套层数的增加，SQL 语句会变得越来越难读懂，性能也会越来越差，因此尽量避免使用多层嵌套的子查询。

## 标量子查询

标量子查询就是返回单一值的子查询，即只返回 1 行 1列的结果。

它可以用在能够使用常数或者列名的地 方，无论是 SELECT 子句、 GROUP BY 子句、 HAVING 子句，还是ORDER BY 子句，几乎所有的地方都可以使用。包括 WHERE 子句，可以规避 WHREE 子句不能使用聚合函数的情况。

如：

```mysql
SELECT product_id, product_name, sale_price
FROM Product
WHERE sale_price > (SELECT AVG(sale_price)
	FROM Product);
```

注意：

绝对不能返回多行结果。

## 关联子查询

由于标量子查询只能返回单行数据，所以在细分的组内进行比较时，标量子查询并不使用，而需要使用关联子查询。

关联子查询就是在子查询中多了一条WHERE语句的条件。

如下：

```mysql
SELECT product_type, product_name, sale_price
FROM Product AS P1
WHERE sale_price > (
    SELECT AVG(sale_price)
	FROM Product AS P2
	WHERE P1.product_type = P2.product_type
	GROUP BY product_type
);
```

# 集合运算

集合最重要的性质之一就是去重，使用 UNION 可以实现多张表的并集（加法）运算。而 Mysql 目前还并不支持差集和补集的运算。

如：

```mysql
select prod_id from productnotes 
union 
select prod_id from products;
```

注：

+   作为运算对象的记录的列数必须相同
+   作为运算对象的记录中列的类型必须一致
+   可以使用任何 SELECT 语句,但 ORDER BY 子句只能在最后使用一次
+   在集合运算符中使用 ALL 选项,可以保留重复行。(`Union All`)



# 表联结

## Where 表联结

集合运算是添加行的运算，而联结就是添加列的运算。联结就是讲根据多张表的字段进行整合和关联，合到一张表中。


```mysql
# 创建联结，规定要联结的所有表以及他们如何关联即可
mysql> select vend_name, prod_name, prod_price
    -> from vendors, products
    -> where vendors.vend_id = products.vend_id
    -> order by vend_name, prod_name;
    
# 联结多个表
mysql> select prod_name, vend_name, prod_price, quantity
    -> from orderitems, products, vendors
    -> where products.vend_id = vendors.vend_id
    -> and orderitems.prod_id = products.prod_id
    -> and order_num = 20005;
```
注意：
1. 应使用完全限定列名（< 表的别名 > . < 列名 >）
2. 应保证所有联结都有 WHERE 子句
3. 要注意考虑性能。不要联结不必要的表，联结的表越多，性能下降越厉害。
5. 表别名只在查询执行中使用，与列别名不一样，表别名不返回到客户机。

## 内联结 - INNER JOIN

实际上WHERE的联结也就是内联结，还有另一种使用 inner join 的内联结方式。

该方式必须使用 ON 子句，并且要书写在 FROM 和 WHERE 之间。

```mysql
# 明确内部联结
select v.vend_name, p.prod_name, p.prod_price
from vendors as v inner join products as p
on v.vend_id = p.vend_id;
```

相比较使用where的联结，内联结明确使用的联结条件，但有时候会影响性能。

## 外联结- OUTER JOIN

内联结只能选取出同时存在于两张表中的数据，相反，对于外联结来说，只要数据存在于某一张表当中，就能够读取出来。

外联结中使用 LEFT 、 RIGHT 来指定主表。使用 LEFT 时 FROM 子句中写在左侧的表是主表，使用 RIGHT 时右侧的表是主表。

# 数据更新

## 插入

INSERT 语法如下：

```mysql
INSERT INTO <表名> (列1, 列2, ...) VALUES (值1, 值2, ...);
```

字段数和值数必须保持一致，但是使用默认值时列数无需完全一致。

如：

```mysql
INSERT INTO ProductIns 
(product_id, product_name, product_type, sale_price, purchase_price, regist_date)
VALUES ('0001', 'T 恤衫 ', ' 衣服 ', 1000, 500);
```

注：

+   对表进行全列 INSERT 时,可以省略表名后的列清单。这时 VALUES 子句的值会默认按照从左到右的顺序赋给每一列。

+   想给某一列赋予 NULL 值时,可以直接在 VALUES子句的值清单中写入 NULL 。

+   省略 INSERT 语句中的列名,就会自动设定为该列的默认值(没有默认值时会设定为 NULL )。

## 删除

 DROP TABLE 语句可以将表完全删除，而 DELETE 语句则会保留表的结构（容器），只是删除全部数据。

```mysql
DELETE FROM <表名>;

# 指定删除对象的 DELETE 语句(搜索型DELETE)
DELETE FROM <表名> WHERE <条件>;
```

注：

+   DELETE 语句的删除对象并不是表或者列，而是记录(行)。

+   DELETE 语句中不能使用 GROUP BY 、HAVING 和 ORDER BY 三类子句,而只能使用 WHERE 子句。

除了 DELETE 以外，Mysql 还有一个 TRUNCATE 语句用来删除。

```mysql
TRUNCATE <表名>;
```

它只能删除全部数据，而不能通过WHERE 子句指定条件来删除部分数据，但是也正因为如此，其处理速度要快得多。

## 更新

使用 UPDATE 语句来更新已有的数据。

```mysql
UPDATE <表名> SET <列名> = <表达式>;

# 指定条件的 UPDATE 语句(搜索型 UPDATE )
UPDATE <表名> SET <列名> = <表达式> WHERE <条件>;
```

使用 UPDATE 语句可以将值清空为 NULL (但只限于未设置 NOT NULL 约束的列)。

# 事务

事务是需要在同一个处理单元中执行的一系列更新处理的集合。

使用事务开始语句和事务结束语句,将一系列 DML 语句( INSERT/UPDATE/DELETE 语句)括起来,就实现了一个事务处理。

```mysql
START TRANSACTION;
	DML 语句1 ;
	DML 语句2 ;
	DML 语句3 ;
事务结束语句(COMMIT 或者 ROLLBACK) ;
```

COMMIT 是提交事务包含的全部更新处理的结束指令，相当于文件处理中的覆盖保存。一旦提交，就无法恢复到事务开始前的状态了。

ROLLBACK 是取消事务包含的全部更新处理的结束指令，相当于文件处理中的放弃保存。一旦回滚，数据库就会恢复到事务开始之前的状态。通常回滚并不会像提交那样造成大规模的数据损失。

# 条件判断

## IF 语句

语法格式如下：

```mysq
IF(expr1,expr2,expr3)
```

若 expr1 为真(expr1 <> 0 且 expr1 <> NULL)，那么返回 expr2，否则返回 xpr3

例如：

```mysql
select if(id, 1, null) from customers;
```

还有个 IFNULL() 函数用来替换 NULL 值，若 value1 不为null ，则返回 value1，否则返回value2

```mysql
IFNULL(value1,value2)
```

## IF..ELSE...

IF...ELSE...做为流程控制语句使用，语法格式如下：

```mysql
IF search_condition THEN 
    statement_list  
[ELSEIF search_condition THEN]  
    statement_list ...  
[ELSE 
    statement_list]  
END IF
```

search_condition是一个条件表达式，可以由“=、<、<=、>、>=、!=”等条件运算符组成，并且可以使用AND、OR、NOT对多个表达式进行组合。

## CASE..WHEN..

句法一：

```mysql
CASE expression
WHEN value1 THEN returnvalue1
WHEN value2 THEN returnvalue2
WHEN value3 THEN returnvalue3
……
ELSE defaultreturnvalue
END
```

句法二：

```mysql
CASE
WHEN condition1 THEN returnvalue1
WHEN condition 2 THEN returnvalue2
WHEN condition 3 THEN returnvalue3
……
ELSE defaultreturnvalue
END
```

# 存储过程

存储过程的定义包含 in、out 和 inout 三种参数。IN 声明的是外界传递给存储过程的参数，OUT 声明的是传递给外界的参数，INOUT 的则对过程的传入和传出。

给变量赋值都需要用 select into 语句。

mysql 中的变量要以 @ 开头。

```mysql
# 创建存储过程，无参数
create procedure procedure_1()
	begin
		select now();
	end

# 创建存储过程，有参数
create procedure procedure_2( in time date, out num int)
    begin
        select count(*) from orders
        where order_date < time
        into num;
    end
    

# 调用存储过程
call procedure_1();
call procedure_2("2018-01-01", @num);
select @num;
```

注：

+   每次只能给一个变量赋值，不支持集合的操作。

+   命令行中创建存储过程需要自定义分隔符，因为命令行是以 ; 为结束符，而存储过程中也包含了分号，因此会错误把这部分分号当成是结束符，造成语法错误。

在命令行定义存储过程：

```mysql
delimiter //

create procedure procedure_1()
	begin
		select now();
    end //
    
delimiter ;  
```

delimiter 用于声明语句的分隔符，第一个声明 `//` 作为新的分隔符，第二个则恢复分隔符 `;` 。

```mysql
# 删除存储过程
drop procedure procedure_1;
```



# 数据类型

## 整型

|           类型           | 占用的存储空间（单位：字节） | 无符号数取值范围 | 有符号数取值范围 |      含义      |
| :----------------------: | :--------------------------: | :--------------: | :--------------: | :------------: |
|        `TINYINT`         |              1               |     0 ~ 2⁸-1     |    -2⁷ ~ 2⁷-1    |  非常小的整数  |
|        `SMALLINT`        |              2               |    0 ~ 2¹⁶-1     |   -2¹⁵ ~ 2¹⁵-1   |    小的整数    |
|       `MEDIUMINT`        |              3               |    0 ~ 2²⁴-1     |   -2²³ ~ 2²³-1   | 中等大小的整数 |
| `INT`（别名：`INTEGER`） |              4               |    0 ~ 2³²-1     |   -2³¹ ~ 2³¹-1   |   标准的整数   |
|         `BIGINT`         |              8               |    0 ~ 2⁶⁴-1     |   -2⁶³ ~ 2⁶³-1   |     大整数     |

## 浮点型

|   类型   | 占用的存储空间（单位：字节） |     绝对值最小非0值      |     绝对值最大非0值      |     含义     |
| :------: | :--------------------------: | :----------------------: | :----------------------: | :----------: |
| `FLOAT`  |              4               |     ±1.175494351E-38     |     ±3.402823466E+38     | 单精度浮点数 |
| `DOUBLE` |              8               | ±2.2250738585072014E-308 | ±1.7976931348623157E+308 | 双精度浮点数 |

### 设置最大和最小位数

```sql
FLOAT(M, D)
DOUBLE(M, D)
```

+   M为最大有效数字的位数（十进制）
+   D为最大小数点后的位数（十进制）

如：

```sql
FLOAT(4, 1) ==> -999.9~999.9
```

## 定点数

|      类型       | 占用的存储空间（单位：字节） |  取值范围  |
| :-------------: | :--------------------------: | :--------: |
| `DECIMAL(M, D)` |          取决于M和D          | 取决于M和D |

M默认是10，D默认为0，M为1-65，D为0-30，且M>=D

## 日期和时间

`MySQL5.6.4`之后的版本支持了小数秒级（毫秒和微妙），存储位数有所变化。

|    类型     |      存储空间要求      |                           取值范围                           |     含义     |
| :---------: | :--------------------: | :----------------------------------------------------------: | :----------: |
|   `YEAR`    |         1字节          |                          1901~2155                           |    年份值    |
|   `DATE`    |         3字节          |                 '1000-01-01' ~ '9999-12-31'                  |    日期值    |
|   `TIME`    | 3字节+小数秒的存储空间 |         '-838:59:59[.000000]' ~ '838:59:59[.000000]'         |    时间值    |
| `DATETIME`  | 5字节+小数秒的存储空间 | '1000-01-01 00:00:00[.000000]' ～ '9999-12-31 23:59:59'[.999999] | 日期加时间值 |
| `TIMESTAMP` | 4字节+小数秒的存储空间 | '1970-01-01 00:00:01[.000000]' ～ '2038-01-19 03:14:07'[.999999] |    时间戳    |

| 保留的小数秒位数 | 额外需要的存储空间要 |
| :--------------: | :------------------: |
|        0         |        0字节         |
|       1或2       |        1字节         |
|       3或4       |        2字节         |
|       5或6       |        3字节         |

基本类型后面加个括号，表示支持的小数秒的位数，如TIME(3)表示精确到毫秒

## 字符型

+   utf8：3个字节
+   **utf8md4**：4个字节（实际的utf8）

|     类型     |   最大长度   |   存储空间要求    |       含义       |
| :----------: | :----------: | :---------------: | :--------------: |
|  `CHAR(M)`   |   M个字符    |     M×W个字节     | 固定长度的字符串 |
| `VARCHAR(M)` |   M个字符    | L+1 或 L+2 个字节 | 可变长度的字符串 |
|  `TINYTEXT`  | 2⁸-1 个字节  |     L+1个字节     | 非常小型的字符串 |
|    `TEXT`    | 2¹⁶-1 个字节 |    L+2 个字节     |   小型的字符串   |
| `MEDIUMTEXT` | 2²⁴-1 个字节 |     L+3个字节     | 中等大小的字符串 |
|  `LONGTEXT`  | 2³²-1 个字节 |     L+4个字节     |   大型的字符串   |

+   其中`M`代表该数据类型最多能存储的**字符**数量，`L`代表我们实际向该类型的属性中存储的字符串在特定字符集下所占的字节数，`W`代表在该特定字符集下，编码一个字符最多需要的字节数

+   “**某一行包含的所有列中存储的数据大小总和不得超过65535个字节**“这个规定不适用于TEXT类型，对其它类型有效

## 其它类型

+   ENUM
+   SET
+   二进制类型：BIT，BINARY，VARBINARY，TINYBLOB，BLOB，LONGBLOB等



# 其它

+   使用字符串或者日期常数时，必须使用单引号 ( ' ) 将其括起来，最好避免使用双引号，因为在某些模式下双引号有特定的含义

+   用 AS 可以设定别名，设定中文名时要用双引号，而非单引号，而中文作为单元的数据要用单引号
+   条件运算符对 NULL 无效，应该用 `is null` 和 `is not null`。
+   逻辑运算符对 NULL 会产生 不确定(UNKNOWN) ，所以尽量不用 NULL 进行任何运算，即尽量讲 NOT NULL 包含进字段的约束条件 