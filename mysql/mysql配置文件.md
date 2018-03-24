---
title: mysql配置文件
tags: mysql
grammar_cjkRuby: true
---

> 在Mysql根目录下有一个`.ini`，这个文件是Mysql数据库中使用的配置文件

### 客户端配置部分 CLIENT SECTION
```
[client]  
port=3306
[mysql]
default-character-set=utf8
```
属性 | 说明
-- | --
port | 参数表示的是MySQL数据库的端口，默认的端口是3306
default-character-set | 客户端默认的字符集，如果你希望它支持中文，可以设置成gbk或者utf8
password | 设置了password参数的值就可以在登陆时不用输入密码直接进入
socket  | socket 服务套接字, 程序使用套接字链接比使用域名:端口链接要快, 因为这样就不需要协议解析了socket = /tmp/mysql.sock

### 服务端配置部分 SERVER SECTION
```
[mysqld]  
basedir= D:\ProgramFiles\mysql-5.7.14-winx64
datadir=D:\ProgramFiles\mysql-5.7.14-winx64\data
default-storage-engine=INNODB
init_connect='SET NAMES utf8'
character-set-server=utf8  
collation-server=utf8_general_ci  
```

属性 | 说明
-- | --
port | 数据库的端口
basedir | MySQL的安装路径
datadir | 表示MySQL数据文件的存储位置，也是数据库表的存放位置
default-character-set | 表示默认的字符集，这个字符集是服务器端的
default-storage-engine | 默认的存储引擎
sql-mode | 表示SQL模式的参数，通过这个参数可以设置检验SQL语句的严格程度
max_connections | 表示允许同时访问MySQL服务器的最大连接数，其中一个连接是保留的，留给管理员专用的
max_connect_errors | 对于同一主机，如果有超出该参数值个数的中断错误连接，则该主机将被禁止连接。如需对该主机进行解禁，执行：FLUSH HOST。
back_log  | MySQL能有的连接数量。当主要MySQL线程在一个很短时间内得到非常多的连接请求，这就起作用，然后主线程花些时间(尽管很短)检查连接并且启动一个新线程。back_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。
query_cache_size | 表示查询时的缓存大小，缓存中可以存储以前通过select语句查询过的信息，再次查询时就可以直接从缓存中拿出信息。
query_cache_limit | 只有小于此设定值的结果才会被缓冲,此设置用来保护查询缓冲,防止一个极大的结果集将其他所有的查询结果都覆盖
table_cache | 表示所有进程打开表的总数
tmp_table_size | 表示内存中临时表的总数
server-id | 表示是本机的序号为1,一般来讲就是master的意思
open_files_limit  | MySQL打开的文件描述符限制，默认最小1024;当open_files_limit没有被配置的时候，比较max_connections*5和ulimit -n的值，哪个大用哪个，当open_file_limit被配置的时候，比较open_files_limit和max_connections*5的值，哪个大用哪个。
thread_cache_size | 参数表示保留客户端线程的缓存
thread_concurrency  | 此允许应用程序给予线程系统一个提示在同一时间给予渴望被运行的线程的数量. 此值只对于支持 thread_concurrency() 函数的系统有意义( 例如Sun Solaris). 你可可以尝试使用 [CPU数量]*(2..4) 来作为thread_concurrency的值
myisam_max_sort_file_size | 表示MySQL重建索引时所允许的最大临时文件的大小
myisam_sort_buffer_size | 表示重建索引时的缓存大小
key_buffer_size | 表示关键词的缓存大小
read_buffer_size | 表示MyISAM表全表扫描的缓存大小
read_rnd_buffer_size | 表示将排序好的数据存入该缓存中
sort_buffer_size | 表示用于排序的缓存大小
skip-external-locking | 如果是多服务器环境 注释掉这一行，如果是单服务器环境，则将其禁用即可
skip-networking | 不在TCP/IP端口上进行监听.如果所有的进程都是在同一台服务器连接到本地的mysqld,这样设置将是增强安全的方法,所有mysqld的连接都是通过Unix sockets 或者命名管道进行的.
skip-name-resolve | 禁止MySQL对外部连接进行DNS解析,只能用ip进行连接
max_allowed_packet  | 服务所能处理的请求包的最大大小以及服务所能处理的最大的请求大小(当与大的BLOB字段一起工作时相当必要)
binlog_cache_size  | 在一个事务中binlog为了记录SQL状态所持有的cache大小,如果你经常使用大的,多声明的事务,你可以增加此值来获取更大的性能.
max_heap_table_size  | 独立的内存表所允许的最大容量.
join_buffer_size | 此缓冲被使用来优化全联合(full JOINs 不带索引的联合).类似的联合在极大多数情况下有非常糟糕的性能表现,但是将此值设大能够减轻性能影响.
table_open_cache | 打开表缓存数 一般推荐设置为和max_connections一样即可
default_table_type | 当创建新表时作为默认使用的表类型,如果在创建表示没有特别执行表类型,将会使用此值
transaction_isolation | 设定默认的事务隔离级别.可用的级别如下:READ-UNCOMMITTED, READ-COMMITTED, REPEATABLE-READ, SERIALIZABLE
net_buffer_length | 每一个客户端线程都是被关联的,用连接缓冲和结果缓冲, 这两个缓冲开始被设置为net_buffer_length这个变量指定的大小,但是当需要增大的时候,会自动的上升为max_allowed_packet变量所指定的大小, 结果缓冲会收缩到net_buffer_length这个变量指定的大小在之后的每个SQL语句,这个变量一般不需要去改变它, 但是如果你的内存比较小的话,那么你可以设置为你期望的大小, 但是如果语句超过了这个长度,那么这个连接缓冲会自动的增大,最大值可以设置为1MB
bulk_insert_buffer_size  | MyISAM 使用特殊的类似树的cache来使得突发插入(这些插入是,INSERT … SELECT, INSERT … VALUES (…), (…), …, 以及 LOAD DATAINFILE) 更快. 此变量限制每个进程中缓冲树的字节数.设置为 0 会关闭此优化.为了最优化不要将此值设置大于 “key_buffer_size”.当突发插入被检测到时此缓冲将被分配.
log_bin  | log_bin = mysql-bin
binlog_format | binlog_format = mixed
expire_logs_days | 30
log_error | /data/mysql/mysql-error.log #错误日志路径
slow_query_log | slow_query_log = 1
long_query_time | 慢查询时间 超过1秒则为慢查询
slow_query_log_file | slow_query_log_file =/data/mysql/mysql-slow.log slow_query_log_file记录日志路径
lower_case_table_names | 1表示不区分大小写 

### INNODB Specific options 配置

属性 | 说明
-- | --
innodb_additional_mem_pool_size | 附加的内存池，用来存储InnoDB表的内容
innodb_flush_log_at_trx_commit | 设置提交日志的时机，若设置为1，InnoDB会在每次提交后将事务日志写到磁盘上。
innodb_log_buffer_size | 用来存储日志数据的缓存区的大小
innodb_buffer_pool_size | 表示缓存的大小，InnoDB使用一个缓冲池类保存索引和原始数据
innodb_log_file_size | 表示日志文件的大小
innodb_thread_concurrency | 表示在InnoDB存储引擎允许的线程最大数
skip-innodb | 你的MySQL服务包含InnoDB支持但是并不打算使用的话,使用此选项会节省内存以及磁盘空间,并且加速某些部分
innodb_data_file_path  | innodb_data_file_path = ibdata1:10M:autoextend InnoDB 将数据保存在一个或者多个数据文件中成为表空间.如果你只有单个逻辑驱动保存你的数据,一个单个的自增文件就足够好了
innodb_data_home_dir | 设置此选项如果你希望InnoDB表空间文件被保存在其他分区.默认保存在MySQL的datadir中
innodb_file_io_threads | 用来同步IO操作的IO线程的数量
innodb_force_recovery | 如果你发现InnoDB表空间损坏, 设置此值为一个非零值可能帮助你导出你的表.从1开始并且增加此值知道你能够成功的导出表
innodb_log_files_in_group | 在日志组中的文件总数.通常来说2~3是比较好的.
innodb_log_group_home_dir | InnoDB的日志文件所在位置. 默认是MySQL的datadir
innodb_flush_method | InnoDB用来刷新日志的方法.表空间总是使用双重写入刷新方法默认值是 “fdatasync”, 另一个是 “O_DSYNC”
innodb_lock_wait_timeout |在被回滚前,一个InnoDB的事务应该等待一个锁被批准多久.InnoDB在其拥有的锁表中自动检测事务死锁并且回滚事务.
innodb_file_per_table |InnoDB为独立表空间模式，每个数据库的每个表都会生成一个数据空间。独立表空间优点：1．每个表都有自已独立的表空间。2．每个表的数据和索引都会存在自已的表空间中。3．可以实现单表在不同的数据库中移动。4．空间可以回收（除drop table操作处，表空不能自已回收）缺点：单表增加过大，如超过100G 结论：共享表空间在Insert操作上少有优势。其它都没独立表空间表现好。当启用独立表空间时，请合理调整：innodb_open_files
innodb_open_files | 限制Innodb能打开的表的数据，如果库里的表特别多的情况，请增加这个。这个值默认是300
innodb_write(read)_io_threads |innodb使用后台线程处理数据页上的读写 I/O(输入输出)请求,根据你的 CPU 核数来更改,默认是4