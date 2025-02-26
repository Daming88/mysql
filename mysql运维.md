### 日志
* 错误日志: 记录了当mysql服务启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息。  
日志位置：默认存放在/var/log/mysql/error.log
```mysql
# 可通过一下命令获取错误日志文件的存放位置
show variables like '%log_error%';
```
![img.png](/image/img_15.png)
* 二进制日志：记录了所有的DDL(数据定义语言)语句和DML(数据操作语言)语句，但不包含数据查询select、show等语句。
    - 作用：
      1. <font color="red">灾难时的数据恢复</font>
      2. <font color="red">mysql的主从复制。在mysql8版本中，默认二进制日志是开启的</font>
```mysql
# 可通过以下命令获取二进制日志文件的存放位置
show variables like '%log_bin%';
```
### 主从复制
* 概述:  
<font color="blue">主从复制是指将主数据库的DDL和DML操作通过二进制日志传到从库服务器中，然后在从库上对这些日志重新执行(也叫重做)，从而使得从库和主库的数据保持同步。</font>  
mysql支持一台主库同时向多台从库进行复制，从库同时也可以作为其他从服务器的主库实现复制
* 作用: 
  1. <font color="purple">主库出现问题，可以快速切换到从库提供服务。</font>
  2. <font color="purple">实现读写分离，降低主库的访问压力。</font>
  3. <font color="purple">可以在从库中执行备份，可以避免备份期间影响主库服务。</font>
* 原理  
![img.png](/image/img_16.png)  
> 从上图来看，复制分三部：
>> 1、Master主库在事务提交时，会把数据变更记录在二进制日志文件中Binlog中。  
>> 2、从库读取主库的二进制日志文件Binlog，写入到从库的中继日志Relay Log。  
>> 3、slave从库重做中继日志的事件，将改变反应它自己的数据。
* 搭建  
主库：192.168.1.146  
从库：192.168.1.113  
> 配置主库：  
> 1、修改配置文件 /etc/mysql/my.cnf，然后重启数据库服务
> 
```mysql
# mysql服务ID，保证整个集群环境中唯一，取值范围：1-2^32-1，默认为1
server-id = 1
# 是否只读，1表示只读，0表示可读写
read-only = 0
# 忽略的数据，指不需同步的数据库
# binlog-ignore-db = mysql
# 指定同步的数据库
# binlog-do-db = test
```
> 2、登录mysql创建远程连接的账号，并授权主从复制权限.
```mysql
# 创建test用户，并设置密码，该用户可在任意主机连接该MYSQL服务
create user 'test'@'%' identified with mysql_native_password by 'test';
# 为test用户授权主从复制权限
grant replication slave on *.* to 'test'@'%';
```
> 3、查看主库的binlog文件，并获取binlog的位点。
```mysql
show master status ;
```
![img.png](/image/img_17.png)
> 字段含义说明
>> File：从哪个文件开始推送日志文件  
> Position：从哪个位置开始推送日志文件  
> Binlog_Ignore_DB：指定不需要同步的数据库
>  Executed_Gtid_Set：指定需要同步的数据库

> 配置从库：  
> 1、修改配置文件 /etc/mysql/my.cnf，然后重启数据库服务
> 
```mysql
# mysql服务ID，保证整个集群环境中唯一，取值范围：1-2^32-1，默认为1
server-id = 2
# 是否只读，1表示只读，0表示可读写
read-only = 1
```
> 2、登录mysql，配置从库连接主库的信息  
```mysql
# 设置主库配置,mysql8.0.23版本以上
CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.1.146',SOURCE_USER='test',SOURCE_PASSWORD='test',SOURCE_LOG_FILE='binlog.000015',SOURCE_LOG_POS=876;
```
![img.png](/image/img_18.png)
> 3、开启同步操作
```mysql
# 8.0.22之前
start slave; 
# 8.0.22之后
start replica ;
# 查看从库状态  \G 把查询结果切换树表形式
show slave status\G;
show replica status\G;
```

> <font color="red">从库配置中遇到的问题解决方案</font>  
> <font color="red">1、在mysql命令行执行 <font color="purple">start replica</font> 报错</font>    
> <font color="red">mysql> start replica ;</font>    
<font color="red">ERROR 1872 (HY000): Replica failed to initialize applier metadata structure from the repository</font>  
> <font color="red">可尝试执行以下命令：</font>
```mysql
# 1、重置从库配置信息
reset slave;
# 2、重新配置主库的连接信息
CHANGE REPLICATION SOURCE TO SOURCE_HOST='192.168.1.146',SOURCE_USER='test',SOURCE_PASSWORD='test',SOURCE_LOG_FILE='binlog.000015',SOURCE_LOG_POS=876;
# 3、重新启动从库
start replica ;
```

