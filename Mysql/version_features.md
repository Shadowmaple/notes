# 版本变化

## MySQL8.0 对比MySQL5.7 新特性

1. 索引隐藏（invisible index）：将某索引隐藏，查询优化器将不会使用该索引，可以用来测试
2. 默认编码：编码默认值从原来的 latin 编码修改为utf8mb4
3. 设置持久化：在线将设置持久化
4. UUID功能增强：存储格式从CHAR(36)变为VARCHAR(16)，并增加三个新函数：BIN_TO_UUID(), UUID_TO_BIN(), IS_UUID()。
5. 倒序索引（Descending Indexes）：可以设置索引为倒序排列
6. 通用表表达式（Common Table Expressions）
7. ……

> <https://www.tnphost.com/support/difference-mysql-5-7-vs-mysql-8-0-learn-whats-new-in-mysql-8-0/>
