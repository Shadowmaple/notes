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

