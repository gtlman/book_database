# <center>MySQL逻辑分层</center>
## 连接层
负责与Client的连接

##服务层
1. 提供数据库操作的API（SELECT, UPDATE......）
2. 优化client提交的SQL命令

##引擎层
|MyISAM|InnoDB|
|:-:|:-:|
|不支持事务|支持事务|
|不支持外键约束|支持外键约束|
|只有表锁|有表锁和行锁|
|支持全文索引|不支持全文索引|

查看当前引擎：
```
SHOW engines \G;
SHOW variables LIKE %storage_engine%;
```

##存储层
负责存储数据