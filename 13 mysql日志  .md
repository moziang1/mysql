# mysql日志



```mysql
     mysql的日志记录mysql数据库的日常操作和错误信息，日志拥有不同的类型，每个种类存放不同类型的日志，可以从日志中获取，mysql运行情况，用户做的操作，错误信息等
```



## 日志分类

```mysql
1. 二进制日志  # 记录所有更改数据的语句,包含(ddl,dml)
2. 错误日志    #记录mysql在启动，运行停止时发生的问题
3. 查询日志    #记录客户端连接到服务器连接和执行的语句
	#客户端连接到mysql，监控这个用户的所有操作信息
	
4. 慢查询日志  #记录所有超过阈(yu)值的查询语句
		#阈值，相当于临界值(上限)，一旦超过这个临界值，则会记录
```

### 1. 二进制日志

```mysql
#二进制日志，记录mysql数据库的变化，以 一种事务安全的方式进行记录，不会记录没有修改数据的语句，可以用于复制和尽可能的恢复数据，默认二进制日志是不启动的

#并且，重启mysql服务也会开启一个二进制日志
```



#### #1 开启二进制日志(5.6)

vim /etc/my.cnf

```mysql
#在[mysqld]下面写入

log-bin=mysql-bin      #开启二进制日志，并且设置日志前缀
expire_logs_days=10        #设置清除日志的天数
max_binlog_size=500M       #设置每个日志的最大容量
```

重启mysql服务

```mysql
/etc/init.d/mysqld restart
```

#查看

```mysql
cd /usr/local/mysql/data/

#可以查看到有mysql-bin.000001这样的文件


#刷新日志  (每次刷新日志都会生成mysql-bin.xxx日志)
flush logs;



[root@localhost data]# ls
auto.cnf                   localhost.localdomain.pid  mysql-bin.000004  mysql-bin.index
ibdata1                    mysql                      mysql-bin.000005  performance_schema
ib_logfile0                mysql-bin.000001           mysql-bin.000006  ss
ib_logfile1                mysql-bin.000002           mysql-bin.000007  test
localhost.localdomain.err  mysql-bin.000003           mysql-bin.000008


```

#### #2 刷新日志的方式

```mysql
1. 重启mysql
2. 在mysql中使用flush logs
3. 在命令行刷新日志 （mysqladmin -u user -p123.com flush-logs）

```



#### #3 查看二进制日志

```mysql
mysql> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       167 |   #单位字节
| mysql-bin.000002 |       167 |
| mysql-bin.000003 |       167 |
| mysql-bin.000004 |       167 |
| mysql-bin.000005 |       167 |
| mysql-bin.000006 |       167 |
| mysql-bin.000007 |       167 |
| mysql-bin.000008 |       167 |
| mysql-bin.000009 |       120 |
+------------------+-----------+
```

#### #4 查看日志设置

```mysql
mysql> show variables like 'log_%';
+----------------------------------------+-------------------------------------------------+
| Variable_name                          | Value                                           |
+----------------------------------------+-------------------------------------------------+
| log_bin                                | ON         #是否开启二进制日志                       |
| log_bin_basename                       | /usr/local/mysql/data/mysql-bin                 |
| log_bin_index                          | /usr/local/mysql/data/mysql-bin.index           |
| log_bin_trust_function_creators        | OFF                                             |
										#是否允许将存储过程，函数写入到二进制日志中，默认不允许，因为如果进行主从复制的话，会把这些操作复制过去，引起不必要的错误
| log_bin_use_v1_row_events              | OFF                                             |
| log_error                              | /usr/local/mysql/data/localhost.localdomain.err |
										#错误日志路径
| log_output                             | FILE                                            |
										#日志输出格式
| log_queries_not_using_indexes          | OFF                                             |
										#是否记录没有索引的日志
| log_slave_updates                      | OFF                                             |
										#是否允许从库更新
| log_slow_admin_statements              | OFF                                             |
| log_slow_slave_statements              | OFF                                             |
| log_throttle_queries_not_using_indexes | 0                                               |
| log_warnings                           | 1                                               |
+----------------------------------------+-------------------------------------------------+
```

#### #5 查看二进制日志的方式

```mysql
#语法格式
'mysqlbinlog 二进制日志名'

'/*!*/;'  #代表存放的数据
```

案例

```mysql
[root@localhost data]# mysqlbinlog mysql-bin.000001;

/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#200115  9:06:21 server id 1  end_log_pos 120 CRC32 0x99be937c 	Start: binlog v 4, server v 5.6.36-log created 200115  9:06:21 at startup
ROLLBACK/*!*/;
BINLOG '
jWUeXg8BAAAAdAAAAHgAAAAAAAQANS42LjM2LWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAACNZR5eEzgNAAgAEgAEBAQEEgAAXAAEGggAAAAICAgCAAAACgoKGRkAAXyT
vpk=
'/*!*/;


# at 120
#200115  9:09:18 server id 1  end_log_pos 167 CRC32 0xec7c9b19 	Rotate to mysql-bin.000002  pos: 4
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[root@localhost data]# mysqlbinlog mysql-bin.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;

# at 4
#200115  9:06:21 server id 1  end_log_pos 120 CRC32 0x99be937c 	Start: binlog v 4, server v 5.6.36-log created 200115  9:06:21 at startup
ROLLBACK/*!*/;
BINLOG '
jWUeXg8BAAAAdAAAAHgAAAAAAAQANS42LjM2LWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAACNZR5eEzgNAAgAEgAEBAQEEgAAXAAEGggAAAAICAgCAAAACgoKGRkAAXyT
vpk=
'/*!*/;



# at 120
#200115  9:09:18 server id 1  end_log_pos 167 CRC32 0xec7c9b19 	Rotate to mysql-bin.000002  pos: 4
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[root@localhost data]# mysqlbinlog mysql-bin.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!40019 SET @@session.max_insert_delayed_threads=0*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;










# at 4
#200115  9:06:21 server id 1  end_log_pos 120 CRC32 0x99be937c 	Start: binlog v 4, server v 5.6.36-log created 200115  9:06:21 at startup
ROLLBACK/*!*/;
BINLOG '
jWUeXg8BAAAAdAAAAHgAAAAAAAQANS42LjM2LWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAACNZR5eEzgNAAgAEgAEBAQEEgAAXAAEGggAAAAICAgCAAAACgoKGRkAAXyT
vpk=
'/*!*/;




# at 120
#200115  9:09:18 server id 1  end_log_pos 167 CRC32 0xec7c9b19 	Rotate to mysql-bin.000002  pos: 4
DELIMITER ;
# End of log file
ROLLBACK /* added by mysqlbinlog */;
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
[root@localhost data]# pwd
/usr/local/mysql/data

```

#### #6 删除二进制日志

```mysql
reset master     #清空所有二进制日志
purge master logs to '二进制名称'  #删除二进制日志之前的所有二进制日志
purge binary logs before 'date'   #删除指定日志之前的所有二进制日志



#注意不要直接用rm -rf 去删除二进制日志文件
mysql-bin.index   #存放二进制的索引信息
如果直接删除，会导致日志出现问题

```

案例（purge master logs to ）

```mysql
purge master logs to 'mysql-bin.000009';

#查看
mysql> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000009 |       120 |  #清除这个日志之前所有的日志
+------------------+-----------+
1 row in set (0.00 sec)

#发现我们的序号还是9
```

案例（reset）

```mysql
reset master;                       #使用reset则在清除日志的时候，还会调整序号

mysql> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       120 |
+------------------+-----------+
1 row in set (0.00 sec)

```



#### #7 二进制日志恢复数据

```mysql
二进制日志除了进行复制以外，还可以用于数据的恢复，但是不提倡恢复数据全部依靠二进制日志
#二进制日志可能会随时清空，所以只有在服务器没有备份的情况下，才会灾难性恢复日志
```

格式

```mysql
mysqlbinlog [option] 日志文件名 | mysql -u root -p123.com

option 中包含的可选项
--start-position #开始的偏移量
--stop-position #结束的偏移量
--start-date  #开始的日期
--stop-date   #结束的日期
```

##### 案例1-1

```mysql
 reset master;       #先清空所有二进制日志

#模拟创建库表，留下日志，然后进行恢复
create database sjk0115;
use sjk0115;
create table bin_test(id int,name char(30));
drop table bin_test;

#写完后，就生成了创建库表的日志

#先查看日志
mysqlbinlog mysql-bin.000001


从这里开始

# at 4     #默认
# at 120
···
create database sjk0115    '找到了创建库 偏移量属于at 120'
···
# at 223
···
create table bin_test(id int,name char(30))     '找到了创建表偏移量属于at 223'
                                            #我们要恢复数据的节点就是223，开始节点
/*!*/;
··

# at 346                                #我不想要删除表操作，所以到这里截至
····
SET TIMESTAMP=1579053847/*!*/;
DROP TABLE `bin_test` /* generated by server */  '找到了删除表 at 346'


#开始恢复，
mysqlbinlog --start-position=223  --stop-position=346 mysql-bin.000001 | mysql -u root -p123.com

'相当于，把执行的命令重新执行一遍，还会留下日志'
```



##### 案例1-2续（偏移量恢复）

```mysql
#插入数据
insert into bin_test values(1,'hello');
insert into bin_test values(2,'hi');
insert into bin_test values(3,'mysql');

#清除表数据
TRUNCATE table bin_test;

#查看二进制日志
mysqlbinlog mysql-bin.000001


#找到我们插入表数据的操作
[root@localhost data]# mysqlbinlog mysql-bin.000001


···

# at 596        #第一个begin是开始的偏移量位置
···
BEGIN           '看这里，我们的增删改的操作，在二进制日志里是一个事务的情况下保存的begin：commit是一对'
/*!*/;

# at 681
···
insert into bin_test values(1,'hello')
···

# at 799
COMMIT/*!*/;
···
# at 830
BEGIN
···

# at 915
insert into bin_test values(2,'hi')
···

# at 1030
···
COMMIT/*!*/;
# at 1061
···
BEGIN
···

# at 1146
insert into bin_test values(3,'mysql')

# at 1264
COMMIT/*!*/;            


# at 1295                #到删除表数据是结束的位置，上面的commit的要的，因为是事务，需要提交
SET TIMESTAMP=1579054730/*!*/;
TRUNCATE table bin_test



#所以命令是
mysqlbinlog --start-position=596   --stop-position=1295 mysql-bin.000001 | mysql -u root -p123.com

#查看数据恢复
mysql> select * from bin_test;
+------+-------+
| id   | name  |
+------+-------+
|    1 | hello |
|    2 | hi    |
|    3 | mysql |
+------+-------+
3 rows in set (0.00 sec)

```



##### 案例1-3（时间恢复日志）

```mysql
TRUNCATE table bin_test;


#查看日志

[root@localhost data]# mysqlbinlog mysql-bin.000001;

····
# at 2200
#200115 10:12:45 server id 1  end_log_pos 2285 CRC32 0x627d6909 	Query	thread_id=9 exec_time=861	error_code=0
SET TIMESTAMP=1579054365/*!*/;
BEGIN
/*!*/;
# at 2285
#200115 10:12:45 server id 1  end_log_pos 2403 CRC32 0x24e7b5d9 	Query	thread_id=9 exec_time=861	error_code=0
SET TIMESTAMP=1579054365/*!*/;
insert into bin_test values(1,'hello')
/*!*/;
# at 2403
#200115 10:12:45 server id 1  end_log_pos 2434 CRC32 0x1cd7ac20 	Xid = 193
COMMIT/*!*/;
# at 2434
#200115 10:12:53 server id 1  end_log_pos 2519 CRC32 0x1231c8b7 	Query	thread_id=9 exec_time=853	error_code=0
SET TIMESTAMP=1579054373/*!*/;
BEGIN
/*!*/;
# at 2519
#200115 10:12:53 server id 1  end_log_pos 2634 CRC32 0xb6692418 	Query	thread_id=9 exec_time=853	error_code=0
SET TIMESTAMP=1579054373/*!*/;
insert into bin_test values(2,'hi')
/*!*/;
# at 2634
#200115 10:12:53 server id 1  end_log_pos 2665 CRC32 0xcb51b9a4 	Xid = 198
COMMIT/*!*/;
# at 2665
#200115 10:13:01 server id 1  end_log_pos 2750 CRC32 0x02587fc4 	Query	thread_id=9 exec_time=845	error_code=0
SET TIMESTAMP=1579054381/*!*/;
BEGIN
/*!*/;
# at 2750
#200115 10:13:01 server id 1  end_log_pos 2868 CRC32 0xa50081c2 	Query	thread_id=9 exec_time=845	error_code=0
SET TIMESTAMP=1579054381/*!*/;
insert into bin_test values(3,'mysql')
/*!*/;
# at 2868
#200115 10:13:01 server id 1  end_log_pos 2899 CRC32 0x36a13d55 	Xid = 203
COMMIT/*!*/;
# at 2899
#200115 10:28:44 server id 1  end_log_pos 3002 CRC32 0xc1cd4d3c 	Query	thread_id=14exec_time=0	error_code=0
SET TIMESTAMP=1579055324/*!*/;
TRUNCATE table bin_test




#恢复命令
mysqlbinlog --start-date='2001-1-5 10:12:45'  --stop-date='2001-1-5 10:28:44' mysql-bin.000001 | mysql -u root -p123.com

```



### 2.错误日志

```mysql
localhost.localdomain.err #这个文件是存放错误日志的

#错误日志有且仅有一个存在于mysql数据目录下，如果将错误日志删除，mysql可以通过flush logs的方式进行找回，但是里面的数据将会消失
```

#### 开启错误日志

```mysql
vim /etc/my.cnf
#添加
	log-error        #可以不写路径，如果写了路径容易冲突

#重启
/etc/init.d/mysqld restart


```



### 3  查询日志 （监视用户操作）

```
记录客户端登陆的所有信息，包含增删改查
```

#### 开启查询日志

```mysql
vim /etc/my.cnf
#添加
	general_log=on

#重启mysql
/etc/init.d/mysqld restart

#查看
localhost.log   #多了个这么个日志
#查看文件发现没有东西
#我们开一个终端，随便做点操作
#再查看，就相当有一个监控
[root@localhost data]# cat localhost.log
/usr/local/mysql/bin/mysqld, Version: 5.6.36-log (Source distribution). started with:
Tcp port: 0  Unix socket: (null)
Time                 Id Command    Argument
200115 11:10:43	    1 Connect	root@localhost on 
		           1 Query	select @@version_comment limit 1
200115 11:10:55	    1 Query	SELECT DATABASE()
		           1 Init DB	sjk0115
		           1 Query	show databases
		           1 Query	show tables
		           1 Field List	bin_test 
200115 11:11:00	    1 Query	show tables
200115 11:11:13	    1 Query	select * from bin_test

#可以看到用户登陆数据库后的所以操作


```

### 4. 慢查询日志（select）

```mysql
记录所有超过指定阈值(时间)的查询语句select，用来找到查询语句的瓶颈，让运维或DBA去解决

#查询语句慢了，就添加索引
```

#### 开启慢查询日志

```mysql
vim /etc/my.cnf
#添加
	long_query_time=5    #设置慢查询日志的阈值 ,查询语句超过5秒的情况下记录，不设置默认10s
	slow_query_log=1   #开启慢查询日志方法
	
	
/etc/init.d/mysqld restart


#查看文件
localhost-slow.log 
```

案例

```mysql
我们设置一个超过5秒的查询语句

select sleep(5);

#查看文件localhost-slow.log 

[root@localhost data]# cat localhost-slow.log 
/usr/local/mysql/bin/mysqld, Version: 5.6.36-log (Source distribution). started with:
Tcp port: 0  Unix socket: (null)
Time                 Id Command    Argument
/usr/local/mysql/bin/mysqld, Version: 5.6.36-log (Source distribution). started with:
Tcp port: 0  Unix socket: (null)
Time                 Id Command    Argument
# Time: 200115 11:29:12
# User@Host: root[root] @ localhost []  Id:     1
# Query_time: 6.000739  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1579058952;
select sleep(6);

```



```mysql
#查看是否开启慢查询
mysql> show variables like 'log_slow%';
+---------------------------+-------+
| Variable_name             | Value |
+---------------------------+-------+
| log_slow_admin_statements | OFF   |
| log_slow_slave_statements | OFF   |
+---------------------------+-------+
2 rows in set (0.00 sec)

#查看阈值
mysql> show variables like 'long_query_time';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 5.000000 |
+-----------------+----------+
1 row in set (0.00 sec)

```

### 总结

```mysql
慢查询日志也可以通过flush logs的方式进行找回，但是数据不存在，除了二进制日志以外，其余3中日志全为文本文件，因为恢复数据时，二进制文件加载速度要高于文本
```

