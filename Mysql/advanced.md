# 字符集与比较规则

1.  Mysql 基本包含所有字符集

2.  GB2312和GBK是汉字字符集，占2个字节，GBK涵盖GB2312，是升级版

3.  Mysql 中的 utf8 字符集每个字符占3个字节，是阉割版的utf8字符集，真正的utf8是Mysql中的 utf8mb4，占四个字节

4.  查看字符集：

    ```sql
    show character set [like 'utf%']
    show charset [like 'utf%']
    ```

    

5.  比较规则包含有是否区分大小写、是否区分重音等，每个字符可有多个比较规则，且有一个默认的比较规则

6.  比较规则的作用通常体现比较字符串大小的表达式以及对某个字符串列进行排序中

7.  查看比较规则：`show collation like 'utf8%'`

8.  字符集和比较规则有服务器级、数据库级、表级和列级四个级别，均可单独设置`character set [字符集]`，`collation set [比较规则]`

9.  从客户端发送请求时的字符集转换：

    -   客户端使用操作系统字符集编码，服务器对接收的字符串采用 `character_set_client` 代表的字符集进行解码，再使用`character_set_connection`字符集编码
    -   采用`character_set_connection`进行解码，再按照具体的列的字符集进行编码。若两者的字符集相同，则无需进行字符集的转换
    -   将从某个列获取到的字节串从该列使用的字符集转换为`character_set_results`代表的字符集后发送到客户端
    -   客户端使用操作系统的字符集进行解析

10.  最好统一设置`character_set_client`、`character_set_connection`和`character_set_results`这三个系统变量的值为同一个字符集：`set names utf8`



# InnoDB的记录格式

1.  **页**是InnoDB采用的磁盘与内存交互的基本单位，一般为16kb

2.  行格式有 Compact、Redundant、Dynamic、Compressed 四种。Redundant是Mysql5.0之前的老格式，Dynamic是Mysql 5.7之后的默认行格式

3.  查看、指定和修改行格式的语法如下：

    ```sql
    show variables like '%row_format';
    create table 表名 (列的信息) row_format=行格式名称
    alter table 表名 row_format=行格式名称
    ```

4.  Compact 行格式主要分为四个部分：变长字段长度列表、NULL值列表、记录头信息、记录的真实数据。

5.  Redundant行格式主要分为三个部分：字段长度偏移列表、记录头信息、记录的真实数据。

6.  变长字段长度列表。对列中的变长类型（varchar、varbinary、text、blob）字段的实际长度，按列顺序逆序排放。

7.  每个变长字段存储所占空间为1或2个字节。仅当该可变字段允许存储的**最大字节数**超过255，并且真实存储的字节数超过127字节，使用2个字节，否则使用1个字节存储。

8.  变长字符集（如utf8、gbk）下的定长类型（如char）的长度也会被纳入变长字段长度列表中。

9.  变长字段长度列表中只存储值为 **非NULL** 的列内容占用的长度，值为 NULL 的列的长度是不储存的

10.  NULL值列表仅存储允许为NULL的字段，逆序排列。每个字段占一位，仅有0和1。值为NULL时为1，非NULL时为0。实际按照整数个字节进行存储，不满一个字节（8位）时，前面自动用0补满。

11.  记录的真实数据除了自定义的列以外，还包含有隐藏列：`row_id`（作为主键的唯一标识id，占6字节）、`transaction_id`（事务id，占6字节）、`roll_pointer`（回滚指针，占7字节）。其中`row_id`只有当未设置主键且未设置Unique键时才会自动添加。

12.  一个行中的所有列（不包括隐藏列和记录头信息）占用的字节长度加起来不能超过65535个字节（不算Text、Blob类型的字段）

13.  行溢出数据。数据量超过16kb则会出现行溢出，本页中只保留前`768`个字节和20个字节的溢出页面地址，剩下数据分散存储到其他页

14.  不只是 VARCHAR(M) 类型的列，其他的 TEXT、BLOB 类型的列在存储数据非常多的时候也会发生行溢出

15.  相较Compact，Dynamic行格式则是将真实数据的所有字节都存储到其它页中，只在本页保留页地址；Compressed行格式在Dynamic的基础上对页采用压缩算法进行压缩

