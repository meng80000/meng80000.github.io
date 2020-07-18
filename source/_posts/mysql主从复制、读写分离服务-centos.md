---
title: mysql主从复制、读写分离服务(centos)
date: 2020-07-18 23:37:26
tags: 
	- MySQL
categories: 数据库
---

### 一、目标：搭建两台MySQL服务器，一台作为主服务器，一台作为从服务器，实现主从复制

### 二、环境：
- 主数据库： 172.18.0.202
- 从数据库： 172.18.0.169
- yum安装mysql安装可参考：https://www.cnblogs.com/brianzhu/p/8575243.html

### 三、原理：
数据库之所以能进行主从复制，主要是因为二进制文件binlog的存在。多台数据库之间可以通过线程进行通信，从库不断的从主库读取binlog日志并且把内容同步到从库上。

<!-- more -->
![1](1.png)

### 四、配置步骤：

1. **保证两个数据库中的库和数据是一致的；**

***（以下为主数据库）***

2. **在主数据中创建一个同步账号（可不创建使用现有的），如果仅仅为了主从复制创建账号，只需要授予REPLICATION SLAVE权限。**```（此账号是主从复制binlog记录使用的，不要与<从数据库>测试账号混淆）```
```
mysql> GRANT REPLICATION SLAVE ON *.* to 'fei'@'172.18.0.169' identified by '123';
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

```
**说明：**
```
备用：
set global validate_password_mixed_case_count=0; 
set global validate_password_length=3; 
set global validate_password_special_char_count=0; 
set global validate_password_policy=0; 
set global validate_password_number_count=3;
SHOW VARIABLES LIKE 'validate_password%'; 
GRANT REPLICATION SLAVE ON *.* to 'fei'@'172.18.0.169' identified by '123';
```
- 如果报1819错误表示密码太简单了，先运行```set global validate_password_policy=0;```设置完这句以后密码就只判断长度了，运行```set global validate_password_number_count=3;  ```
- 查看添加的用户：
`select user,host from mysql.user;`

3. **配置主数据库**
1）要主数据库，你必须要启用二进制日志（binary logging），并且创建一个唯一的Server ID，这步骤可能要重启MySQL。
2）主服务器发送变更记录到从服务器依赖的是二进制日志，如果没启用二进制日志，复制操作不能实现（主库复制到从库）。
3）复制组中的每台服务器都要配置唯一的Server ID，取值范围是1到(232)−1，你自己决定取值。
4）配置二进制日志和Server ID，你需要关闭MySQL和编辑my.cnf或者my.ini文件，在 [mysqld] 节点下添加配置。
5）下面是启用二进制日志，日志文件名以“master-bin”作为前缀，Server ID配置为1，如下：
```
[mysqld]
log-bin=mysql-bin # 默认在是/var/lib/mysql下
server-id=1 #主库id，必须唯一
binlog-do-db=test # 要同步的库
binlog-ignore-db=mysql #不给从机同步的库(多个写多行)
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema
binlog-ignore-db=sys 
expire_logs_days=7 #自动清理 7 天前的log文件,可根据需要修改
```
4. **重启mysql**
```
systemctl restart mysqld
```
5. **查看主服务器状态：**
```
mysql> show master status;
+-------------------+----------+--------------+------------------+-------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-------------------+----------+--------------+------------------+-------------------+
| master-bin.000003 |     8518 |     test     | mysql            |                   |
+-------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```

**注意：记录好File和Position，后面要用**

***（以下为从数据库）***

6. **配置从数据库：**
1）从服务器，同理，要分配一个唯一的Server ID，需要关闭MySQL，修改好my.cnf后再重启，如下：
```
[mysqld]
server-id = 2 # 集群内唯一
log-bin=salve-bin
relay-log=slave-relay-bin
relay-log-index=slave-relay-bin.index
# read-only=true # 从库只读，但是root依然可以修改，所以需要设置非root账号进行使用

```
2）在从服务器里配置连接主服务器的信息：
进入mysql:
```
mysql> stop slave;

mysql> change master to master_host='172.18.0.202', master_port=3306, master_user='fei', master_password='123', master_log_file='master-bin.000003', master_log_pos=8515;

mysql> start slave;
```
**说明：**
- 172.18.0.202是主服务器的id。
- master_log_file='master-bin.000003'是主服务器的File（你主服务器查出来的是什么就写什么）。
- master_log_pos=8515是主服务器的Position（你主服务器查出来的是什么就写什么）。
- 每次重新启动主服务器，master_log_file和master_log_pos都会变。

3）查看状态
```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.18.0.202
                  Master_User: fei
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master-bin.000003
          Read_Master_Log_Pos: 8518
               Relay_Log_File: slave-relay-bin.000020
                Relay_Log_Pos: 4020
        Relay_Master_Log_File: master-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

```
**说明：**
- Slave_IO_Running: Yes
- Slave_SQL_Running: Yes
都是yes就说明成功了。

4）若 Slave_SQL_Running: no 请重复执行以下内容，直至yes：

```
mysql>stop slave; 
mysql>set GLOBAL SQL_SLAVE_SKIP_COUNTER=1; 
mysql>start slave;
```

5）需要为从库创建一个登录主库的账号

```
create user 'test'@'localhost' identified by '123';
grant select,insert,update,delete on test.* to 'test'@'localhost' identified by '123';
flush privileges; 
```

### 五、测试

- 主库添加信息"fei"
```
mysql> select * from box;
+----+---------+------+
| id | name    | flag |
+----+---------+------+
| 16 | test222 | A    |
| 17 | test111 | A    |
| 18 | fei     | A    |
+----+---------+------+
3 rows in set (0.00 sec)
```
- 从库查看：
```
mysql> select * from box;
+----+------+------+
| id | name | flag |
+----+------+------+
| 18 | fei  | A    |
+----+------+------+

```
**这边我们也发现，因为主从是通过binlog进行同步的，所以在同步之前的数据没有写入到当前log里面，因此也就没有办法自动进行同步了。**

### 六、一主多从

有了主从复制的操作案例。我们进行一主多从的配置时是非常简单的。只需要按照相同的配置，再添加一台从库服务器即可。

**从服务器配置如下：**

```
server-id = 3 # 唯一
read-only=true
```
```
mysql> change master to master_host='172.18.0.171', master_port=3306, master_user='tom', master_password='123', master_log_file='master-bin.000003', master_log_pos=8515;

mysql> start slave;
```
**为从库创建一个登录主库的账号**
```
create user 'tom'@'localhost' identified by '123';
grant select,insert,update,delete on test.* to 'tom'@'localhost' identified by '123';
flush privileges; 
```

搞定，按照主从复制的方法进行测试。当主库进行数据添加的时候，多个从库进行了同步的更新
### 七、读写分离
- 对于mysql单实例数据库和master库，如果需要设置为只读状态，需要进行如下操作和设置：
```
mysql -uroot -p
mysql> show global variables like "%read_only%";
mysql> flush tables with read lock;
mysql> set global read_only=1;
mysql> show global variables like "%read_only%";
```
- 将MySQL从只读设置为读写状态的命令：
```
mysql> unlock tables;
mysql> set global read_only=0;
```
- 对于需要保证master-slave主从同步的```salve库```，如果要设置为只读状态，需要执行的命令为：
```
mysql> set global read_only=1;
```
- 将salve库从只读状态变为读写状态，需要执行的命令是：
```
mysql> set global read_only=0;
```
也可以通过配置文件：
```
read-only=true 
```
- 对于数据库读写状态，主要靠 ```read_only=1```全局参数来设定；默认情况下，数据库是用于读写操作的，所以```read_only```参数也是0或faluse状态，这时候不论是本地用户还是远程访问数据库的用户，都可以进行读写操作；如需设置为只读状态，将该```read_only```参数设置为1或TRUE状态，但设置```read_only=1```状态有两个需要注意的地方：
1. ```read_only=1```只读模式，不会影响slave同步复制的功能，所以在MySQL slave库中设定了```read_only=1```后，通过```show slave status\G```命令查看salve状态，可以看到salve仍然会读取master上的日志，并且在slave库中应用日志，保证主从数据库同步一致；
2. ```read_only=1```只读模式，可以限定普通用户进行数据修改的操作，但不会限定具有super权限的用户（root）的数据修改操作；在MySQL中设置```read_only=1```后，普通的应用用户进行```insert、update、delete```等会产生数据变化的DML操作时，都会报出数据库处于只读模式不能发生数据变化的错误，但具有super权限的用户，例如在本地或远程通过root用户登录到数据库，还是可以进行数据变化的DML操作；

- 为了确保所有用户，包括具有super权限的用户也不能进行读写操作，就需要执行给所有的表加读锁的命令 ```flush tables with read lock;```这样使用具有super权限的用户登录数据库，想要发生数据变化的操作时，也会提示表被锁定不能修改的报错。

- 这样通过 设置```set global read_only=1;```和```flush tables with read lock;```两条命令，就可以确保数据库处于只读模式，不会发生任何数据改变，在MySQL进行数据库迁移时，限定master主库不能有任何数据变化，就可以通过这种方式来设定。

- 但同时由于加表锁的命令对数据库表限定非常严格，```如果再slave从库上执行这个命令后，slave库可以从master读取binlog日志，但不能够应用日志，slave库不能发生数据改变，当然也不能够实现主从同步了，```这时如果使用 ```unlock tables;```解除全局的表读锁，slave就会应用从master读取到的binlog日志，继续保证主从库数据库一致同步。

- **```为了保证主从同步可以一直进行，在slave库上要保证具有super权限的root等用户只能在本地登录，不会发生数据变化，其他远程连接的应用用户只按需分配为select,insert,update,delete等权限，保证没有super权限，```则只需要将salve设定```read_only=1```模式，即可保证主从同步，又可以实现从库只读。**

- 当然设定了```read_only=1```后，所有的select查询操作都是可以正常进行的。
### 八、互为主从

互为主从时，需要注意id步长的问题，每一台只生成一个固定步长的id。这边因为是两台机器，所以步长为2。
A数据库只生产ID为 1 3 5 7 9 。。。 的数据。
B数据库只生产ID为 2 4 6 8 10 。。。。
```
auto-increment-increment = 2 
auto-increment-offset = 100（另外一台配置单数101）
```
刪掉所有日志，reset slave;
再执行flush logs;
可以保证你不会因为binlog的原因导致出错。
此时开启start slave;我们发现
![2](2.png)
两台mysql服务器按照自己预先设置的ID步长进行数据添加，ID不冲突，并且互为主从，互相复制。
### 九、多主多从结构
多主多从结构，其实就是在双主结构上把从库也加入进来。唯一需要注意的：从库只会读取一台主库上的binlog，而每台主库的binlog都是“残缺”的，因此需要使用
```
log-slave-updates=on
```
来促使多台主库之间更新互相之间的binlog，重启服务即可。
### 十、测试与思考

1. 如果从库数据不同步，会出现什么情况？
答：如果从库人为加入一条数据，那么同步就失效了，因此主从架构里从库一定不要写入数据。
2. 如果同步之前，主库就有数据会怎么样？
答：不会自动同步，只会同步binlog里面的数据。所以还是停掉主库（防止数据写入），先进行备份，再进行主从架构配置。
3. 从库怎么设置智能只读？
答：read-only=true可以让从库只读，但是这边依然无法限制root权限的人进行修改和写操作，一般做法是创建一个普通权限的用户，登录的时候用普通权限，这样就只能只读了。但是如果用root依然是可以修改的。
4. 如果主库重启会怎么样？
答：binlog的名称发生了改变。主库重启，从库会自动跟上binlog的位置。
5. 如果从库重启会怎么样？
答：从库会自动跟上binlog的位置。
6. 长时间关机会倒是BInlog更新对不上。（重点）
答：需要到日志目录下（正常是/var/lib/mysql）将所有binlog删除。删除完回到Mysql命令行执行flush logs;刷新日志。必要时reset slave;重置一些从库信息。然后再继续配置主从架构。

```
reset slave;
flush logs;
```
—————————————————————————————
###### 问题一：**```如果配置都没问题，但就是看不到库和表，这里必须自己手动建立一样的库和表，之后就可以看到数据同步了。```**
###### 问题二：当使用 grant 权限列表 on 数据库 to ‘用户名’@’访问主机’ identified by ‘密码’; 时会出现”……near ‘identified by ‘密码” at line 1”这个错误
- 因为新版的的mysql版本已经将创建账户和赋予权限的方式分开了
- 解决办法:创建账户:```create user ‘用户名’@’访问主机’ identified by ‘密码’;```赋予权限:```grant 权限列表 on 数据库 to ‘用户名’@’访问主机’  with grant option;```
```
create user 'win'@'localhost' identified by '123';
grant select,insert,update,delete on test.* to 'win'@'localhost' with grant option;
```
###### 问题三：[Mysql主从同步Slave_IO_Running：Connecting ; Slave_SQL_Running：Yes的情况故障排除](https://blog.csdn.net/mbytes/article/details/86711508)


参考：https://blog.csdn.net/csdn317797805/article/details/100932662
参考：https://blog.csdn.net/zleiw/article/details/78243316
