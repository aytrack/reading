## Greenplum基本操作

### 系统管理
#### 启动
通过`gpstart`来启动已经通过`gpinitsystem`初始化过的并且通过`gpstop`停止的greenplum数据库，此命令启动所有greenplum集群中的Postgresql实例。启动各进程是并行。
在master节点执行命令：  
```bash
$ gpstart 
```

#### 重启
```bash
$ gpstop -r
``` 

#### 重新加载配置文件
重新加载master节点上的`pg_hpa.conf`和`postgresql.conf`文件
```bash
$ gpstop -u 
```

#### 停止集群
```bash
$ gpstop                #当有客户端连接时无法停止
$ gpstop -M faster      #停止之前自动回滚所有进程并关闭客户端连接
```

#### 停止客户端
可以使用超级管理员权限来取消或者停止客户端连接进程。
- pg_cancel_backend()
  - pg_cancel_backend( pid int4 )
  - pg_cancel_backend( pid int4, msg text )
- pg_terminate_backend()
  - pg_terminate_backend( pid int4 )
  - pg_terminate_backend( pid int4, msg text )
查询连接进程的命令如下：  
```sql
=# SELECT usename, procpid, waiting, current_query, datname
     FROM pg_stat_activity;
```
通过查询语句完成取消或者停止操作，成功返回`true`,失败返回`false`,如下：
```sql
=# SELECT pg_cancel_backend(31905 ,'Admin canceled long-running query.');
ERROR:  canceling statement due to user request: "Admin canceled long-running query."
```

### 数据库连接
连接greenplum和连接postgresql一样，使用psql从master节点进行连接即可。
使用psql连接：
```bash
$ psql -d gpdatabase -h master_host -p 5432 -U gpadmin

$ psql gpdatabase

$ psql

$ psql postgres         #要连接的数据库还未被创建，可以连接入postgres数据库
```

### 数据库配置
master和每个segment都有自己的postgresql.conf文件，
local类型参数： 是本地的，需要在每一个节点上去修改
master： 只需要在master上更改，当segment连接的时候会自动读取master的配置。

#### local配置参数
使用gpconfig命令修改参数可以同步修改个segment的postgresql.conf文件
```bash
$ gpconfig -c gp_vmem_protect_limit -v 4096
$ gpstop -r
```

### 数据库操作
#### 创建数据库
默认情况下，所有数据库的创建clone自默认的数据库模板`template1`  
```sql
=> CREATE DATABASE new_dbname;
```
```bash
$ createdb -h masterhost -p 5432 mydatabase
```

#### clone数据库
```sql
=> CREATE DATABASE new_dbname TEMPLATE old_dbname;
```

#### 创建其他用户的数据库
```sql
=> CREATE DATABASE new_dbname WITH owner=new_user;
```

#### 查询数据库列表
```sql
=> \l                                   --psql客户端连接时
=> SELECT datname from pg_database;     --其他程序客户端
```

#### 修改数据库
只有owner和超级管理员才能操作。
```sql
=> ALTER DATABASE mydatabase SET search_path TO myschema, public, pg_catalog;
```

#### 删除数据库
删除数据库时会删除系统catalog entries并且删除硬盘上对应的数据文件夹，当有客户端连接时不能删除。
```sql
=> \c postgres
=> DROP DATABASE mydatabase;
```
```bash
$ dropdb -h masterhost -p 5432 mydatabase
```

