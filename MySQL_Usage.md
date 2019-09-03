# <center>MySQL使用相关</center>
## MySQL常用类型
### 字符串/文本
1. **CHAR**（定长字符串），0-255字节
2. **VARCHAR**（变长字符串），0-65535（2^16）字节
3. **TEXT**（长文本），0-65535（2^16）字节
4. **BLOB**（二进制长文本数据），0-65535（2^16）字节

### 数值
1. **INT**（整型）->4字节，unsigned：（0，2^32），signed：（-2^31，2^31）
2. **BIGINT**（长整型）->8字节，unsigned：（0，2^64），signed：（-2^63，2^63）
3. **FLOAT**（单精度浮点）->4字节32位=1符号23(尾数) * 8( 基数)的E(指数）
4. **DOUBLE**（双精度浮点）->8字节64位=1符号52(尾数) * 11( 基数)的E(指数）

### 日期&时间
1. **Date**：YYYY-MM-DD（1000-01-01/9999-12-31）
2. **Time**：HH:MM:SS（'-838:59:59'/'838:59:59'）
3. **Year**：YYYY（1901/2155）
4. **DATETIME**：YYYY-MM-DD HH:MM:SS
5. **TIMESTAMP**：YYYYMMDD HHMMSS（1970-01-01 00:00:00/2038）

## MySQL常用连接
__内连接INNER JOIN__：返回两个表都符合匹配规则的查询结果

`SELECT * FROM TABLE_A AS A INNER JOIN TABLE_B AS B ON A.id=x AND B.id=y;`

__外连接LEFT/RIGHT (OUTER可省略) JOIN__：左外连接返回作表全部，右表不匹配的字段设NULL，右外连接同理。与内连接不同之处在于，两个表之一是完整的。

`SELECT * FROM TABLE_A AS A LEFT JOIN TABLE_B AS B ON A.id=B.id;`

__全连接FULL JOIN__：LEFT JOIN 加上 RIGHT JOIN

`SELECT * FROM TABLE_A AS A FULL JOIN TABLE_B AS B ON A.id=B.id;`

## 乐观锁和悲观锁
### 悲观锁
- 对数据的并发操作持悲观态度

- 先取锁在操作，可以确保数据独占和正确性，但会造成阻塞，例如MySQL中的一锁二查三更新（SELECT FOR UPDATE），行/表/读/写锁等等独占锁

### 乐观锁
- 对数据的并发操作持乐观态度

- 通过记录数据的时间戳/版本号，在提交更新时校对，如果中途有修改则回退操作取最新值重新更新。无需加锁也就不会阻塞，而且单次操作的性能较好，但可能失败。例如版本号机制，CAS算法以及数据库MVCC

***根据数据操作的速度，冲突频率，重试代价来决策***
- 悲观锁主要用在高并发下数据写操作激烈的环境，以及使用锁的成本低于回滚事务的成本时

- 乐观锁适用在写操作较少的情况，节省加锁的开销能提高整体性能

### 版本号机制
在表中加入版本号version字段，修改时先查询version值，修改后更新version+1

### CAS算法（compare and swap）
不使用锁实现多线程之间的变量同步，即非阻塞同步。算法实现需要三个操作量：

1. 需要进行读写的内存对应的数值V
2. 预期值A
3. 拟写入值B

仅当V=A的时候，修改V为B

### MVCC（multi-version concurrent control）多版本并发控制
MVCC允许数据具有多个版本，版本可能使用时间戳或者全局递增事务ID表示，实现同一时刻不同事务查询到的数据可以是不同的。（保证数据符合事务的逻辑顺序）

### MVCC的innodb实现
1. 写操作（DELECT，UPDATE，INSERT）会增加全局事务版本号
2. INSERT会在插入数据的时候设置创建版本号
3. DELECT跟新数据的删除版本号
4. UPDATE操作转换为INSERT+DELECT，也就是保留旧版本数据的同时插入新版本数据。因此innodb需要定期清理。
5. SELECT查询条件以下（当前事务查询的数据  已经创建还未删除）
    1. 创建版本号 低于当前事务版本
    2. 删除版本号 高于当前事务版本