# <center>MySQL与Python的交互</center>
## <center></center>
因为 Django 推荐使用 **[mysqlclient](https://pypi.org/project/mysqlclient/ "mysqlclient")** 驱动，而我后面还要写 关于 Django-ORM 的使用，因此本篇基于 **[mysqlclient](https://pypi.org/project/mysqlclient/ "mysqlclient")** 

官方提供的驱动是 **[mysql-connector](https://pypi.org/project/mysql-connector-python/ "mysql-connector")**


## <center>安装</center>
首先需要安装 python3 和 mysql 的开发环境
```
sudo apt-get install python3-dev default-libmysqlclient-dev  # Debian / Ubuntu
sudo yum install python3-devel mysql-devel                   # Red Hat / CentOS
```

然后从 PyPI 安装 mysqlclient: `pip install mysqlclient`


## <center>使用</center>
***[MySQLdb官方文档](https://mysqlclient.readthedocs.io/user_guide.html#mysqldb "MySQLdb官方文档")***

### MySQLdb.connect() 参数
连接数据库并且返回 Connection 对象
```
host                  -> str        # 目标机器名/地址
user                  -> str        # 用户验证账号
passwd                -> str        # 用户验证密码
db                    -> str        # 需要连接的数据库名
port                  -> int        # MySQL 服务器端口，默认3306
unix_socket           -> str        # UNIX socket 文件的位置
conv                  -> dict       # 自定义类型转换方式,例如默认 int 会调用 Thing2Str() 转换为 字符串再 传给 int 类
compress              -> bool       # 协议压缩???
connect_timeout       -> int        # 设置连接超时时间
named_pipe            -> str        # 与一个命名管道相连接，仅限 windows 操作系统
init_command          -> str        # 当连接创建后马上执行该命令
read_default_file     -> str        # 使用指定的MySQL配置文件
read_default_group    -> str        # 设置使用配置文件的组
cursorclass           -> type       # cursor()使用的类，默认 MySQLdb.cursors.Cursor，要求作为关键字参数
use_unicode           -> bool       # python3默认是True（奇怪MySQLdb不是不支持python3吗？）？？？？，要求作为关键字参数
charset               -> str        # ???，要求作为关键字参数
sql_mode              -> str        # SQL session的模式会改变???，要求作为关键字参数
ssl                   -> dict       # 包含了 SSL 连接参数，如果客户端不支持 SSL，会抛出 NotSupportedError 异常，要求作为关键字参数
```

### Connection 对象
```
cursor([cursorclass])               # 创建该连接的光标，通过光标执行SQL，并获取返回给客户端的结果集，Cursor 返回 tuple, DictCursor 返回 dict, SSCursor->Cursor的服务器版，SSDictCursor->DictCursor的服务器版：服务器端光标的结果集存放在服务端，在使用的时候一行行拿取，只有在结果集很大的时候才需要用到
commit()                            # 如果支持事务则提交事务, 否则啥也不干
rollback()                          # 如果支持事务则回滚事务，否则抛出 NotSupportedError 异常
```

### Cursor 对象
```
execute(self, query, args=None)     # 执行SQL，并且返回 rowcount : SQL影响的行数量
c=db.cursor()
max_price=5
creator='yan'
condition_dict = {'max_price':5, 'creator':'yan'}
1. c.execute("""SELECT spam, eggs, sausage FROM breakfast WHERE price < %s and creator = %s""", (max_price,creator))                # 当传入的是一个list，tuple这类序列时用 %s
2. c.execute("""SELECT spam, eggs, sausage FROM breakfast WHERE price < %(max_price)s and creator = %(creator)s""", condition_dict) # 当传入的是一个dict这类映射时用 %(key)s

executemany(self, query, args)      # 执行同时插入多行数据这类的多行操作, 并且返回 rowcount : SQL影响的行数量
c=db.cursor()
c.executemany(
      """INSERT INTO breakfast (name, spam, eggs, sausage, price)
      VALUES (%s, %s, %s, %s, %s)""",
      [
      ("Spam and Sausage Lover's Plate", 5, 1, 8, 7.95 ),
      ("Not So Much Spam Plate", 3, 2, 0, 3.95 ),
      ("Don't Wany ANY SPAM! Plate", 0, 4, 3, 5.95 )
      ] )                           # 插入三行数据

fetchall(self)                      # 返回结果集，可迭代对象，因为内部实现了__iter__方法，因此直接迭代 Cursor 光标对象也能得到一样的结果，总的来说并没有什么卵用
c.execute("""SELECT * FROM df_goods;""")
res = c.fetchall()
for i in res:
    print(i)
for i in c:                         # 内部通过 fetchone() 实现了__iter__方法
    print(i)                        

close(self)                         # 关闭光标，对于服务器端光标尤其要记得用完关闭，推荐可以使用 with 关键字
```