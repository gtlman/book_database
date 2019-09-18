# <center>MySQL使用相关</center>


## <center>MySQL常用类型</center>
### 字符串/文本
1. **CHAR**（定长字符串），0-255字节
2. **VARCHAR**（变长字符串），0-65535（2^16）字节
3. **TEXT**（长文本），0-65535（2^16）字节
4. **BLOB**（二进制长文本数据），0-65535（2^16）字节

### 数值
1. **TINYINT**（微整型）->1字节，unsigned：（0,255），signed：（-128,127）
2. **INT**（整型）->4字节，unsigned：（0，2^32），signed：（-2^31，2^31）
3. **BIGINT**（长整型）->8字节，unsigned：（0，2^64），signed：（-2^63，2^63）
4. **FLOAT**（单精度浮点）->4字节32位=1符号23(尾数) * 8( 基数)的E(指数）
5. **DOUBLE**（双精度浮点）->8字节64位=1符号52(尾数) * 11( 基数)的E(指数）
6. **BIT(M)**（二进制数据类型）->(M+7)/8 字节   ex:BIT(2)->[00,11], BIT(3)->[000,111]

### 日期&时间
1. **Date**：YYYY-MM-DD（1000-01-01/9999-12-31）
2. **Time**：HH:MM:SS（'-838:59:59'/'838:59:59'）
3. **Year**：YYYY（1901/2155）
4. **DATETIME**：YYYY-MM-DD HH:MM:SS
5. **TIMESTAMP**：YYYYMMDD HHMMSS（1970-01-01 00:00:00/2038）

### 枚举类型
**ENUM**：MySQL会创建枚举值的hash表，数据表中存的是对应索引->（1或者2个字节，取决于枚举值的个数），最多可以有65535个枚举值 ex: enum('a','b','c')

## <center>数据类型使用策略</center>
### TINYINT vs BIT vs ENUM
类似性别这类字段，使用 TINYINT 存 0,1,2

然后在服务端将数据转换为男,女,未知

### CHAR vs VARCHAR
当表大小小于Innodb buffer pool时，CHAR和VARCHAR没有差别，而在表大小大于Innodb buffer pool时，VARCHAR性能反而更高！

当Innodb buffer pool小于表大小时，"磁盘读写"成为了性能的关键因素，而VARCHAR更短，因此性能反而比CHAR高。

优先使用VARCHAR，特别是字符串的平均长度比最大长度要小很多的情况

如果你的字符串本来就很短，例如只有10个字符，那么就优先选CHAR了

### FLOAT vs DOUBLE vs DECIMAL
1. 如果你要表示的浮点型数据转成二进制之后能被32位float存储或者可以容忍截断(二进制小数可能出现无限循环)，则使用float，这个范围大概为要精确保存6位数字左右的浮点型数据，比如10分制的店铺积分可以用float存储，小商品零售价格(1000块之内)
2. 如果你要表示的浮点型数据转成二进制之后能被64位double存储，或者可以容忍截断，这个范围大致要精确到保存13位数字左右的浮点型数据比如汽车价格,几千万的工程造价
3. 相比double，已经满足我们大部分浮点型数据的存储精度要求，如果还要精益求精，则使用decimal定点型存储比如一些科学数据，精度要求很高的金钱


## <center>MySQL常用连接</center>
__内连接INNER JOIN__：返回两个表都符合匹配规则的查询结果

`SELECT * FROM TABLE_A AS A INNER JOIN TABLE_B AS B ON A.id=x AND B.id=y;`

__外连接LEFT/RIGHT (OUTER可省略) JOIN__：左外连接返回作表全部，右表不匹配的字段设NULL，右外连接同理。与内连接不同之处在于，两个表之一是完整的。

`SELECT * FROM TABLE_A AS A LEFT JOIN TABLE_B AS B ON A.id=B.id;`

__全连接FULL JOIN__：LEFT JOIN 加上 RIGHT JOIN

`SELECT * FROM TABLE_A AS A FULL JOIN TABLE_B AS B ON A.id=B.id;`

## <center>乐观锁和悲观锁</center>
### 悲观锁
- 对数据的并发操作持悲观态度

- 先取锁在操作，可以确保数据独占和正确性，但会造成阻塞，例如MySQL中的一锁二查三更新（SELECT FOR UPDATE），行/表/读/写锁等等独占锁

#### InnoDB 的 行锁 和 表锁
MySQL的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同行的记录，但是如果是使用相同的索引键，是会出现锁冲突的，同理就是在没有索引的情况下，InnoDB只能使用表锁。当我们给其增加一个索引后，InnoDB才会锁定符合条件的行

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