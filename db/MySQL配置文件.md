## [client]
```shell
port = 3306
```
## [mysql]
```shell
default-character-set = utf8
```
## [mysqld]
```shell
# 设置时区
default-time-zone = '+8:00'

# 设置最大连接数
max_connections = 100
# 缓存多少线程给客户端使用
thread_cache_size = 8
# 多少个连接请求可以被存在堆栈中
back_log = 100

# 端口
port = 3306

# 字符集
character-set-server = utf8mb4

# 数据文件夹
datadir = '/data'

# 错误日志
log-error = /var/lib/mysql/mysqld.log

# 默认存储引擎
default-storage-engine = MyISAM

# 等待超时时间
wait_timeout = 31536000

# 计划任务
event_scheduler = ON

# 单个查询SQL的大小限制
max_allowed_packet = 128M

# group_concat 长度限制
group_concat_max_len = 1638400

# 对不更新的表，使用查询缓存
query_cache_type = ON
# 在查询常变，表结构常变的情况下，用缓存反而不好；此设置指定单个查询能够使用的缓冲区大小，存可设置为 10M
query_cache_size = 0

# 高速缓存表的数量
table_open_cache = 32

# 单张临时表的大小限制
tmp_table_size = 26M
```
### MYISAM
效率高，不适合大量并发操作。

Mysql数据库中MyISAM表在读操作占主导的应用中是很高效的。可一旦出现大量的读写并发，同InnoDB相比，MyISAM的效率就会直线下降，而且，MyISAM和InnoDB的数据存储方式也有显著不同：
- 通常，在MyISAM里，新数据会被附加到数据文件的结尾，可如果时常做一些UPDATE，DELETE操作之后，数据文件就不再是连续的，形象一点来说，就是数据文件里出现了很多空洞，此时再插入新数据时，按缺省设置会先看这些空洞的大小是否可以容纳下新数据，如果可以，则直接把新数据保存到其中，反之，则把新数据保存到数据文件的结尾。之所以这样做是为了减少数据文件的大小，降低文件碎片的产生。
- 但InnoDB里则不是这样，在InnoDB里，由于主键是cluster的，所以，数据文件始终是按照主键排序的，如果使用自增ID做主键，则新数据始终是位于数据文件的结尾。
```shell
# 并发插入
# 当concurrent_insert=0时，不允许并发插入功能。=1时，允许对没有洞的表使用并发插入，新数据位于数据文件结尾（缺省）。=2时，不管表有没有洞洞，都允许在数据文件结尾并发插入。concurrent_insert设置为2是很有效的，至于由此产生的文件碎片，可以定期使用OPTIMIZE TABLE语法优化。
concurrent_insert = 2

# 重建索引时临时文件允许的最大值
myisam_max_sort_file_size = 100G

# 表发生变化重新排序需要的缓冲大小
myisam_sort_buffer_size = 52M

# MyISAM表索引缓存大小
# 建议不要超过可用内存的30%
key_buffer_size = 1G

# MyISAM表全表扫描时的缓冲区大小（顺序读取）
read_buffer_size = 16M

# 顺序扫描缓冲区
record_buffer = 16M

# 随机读取的缓冲区大小
read_rnd_buffer_size = 256K

# 排序使用的缓冲区大小
# 加速ORDER BY或GROUP BY操作
sort_buffer_size = 16M 
```
### INNODB
```shell
# 禁用innodb
skip-innodb

# 每个表单独一个文件
innodb_file_per_table

# 设置InnoDB存储引擎存放数据字典信息和内部数据结构的内存大小
innodb_additional_mem_pool_size = 4M

# tablespace空间已经满了后，需要MySQL自动扩展多少空间
innodb_autoextend_increment = 128M

# N（N是后面设置的值）次事务提交或事务外的指令就需要把日志写入硬盘
# 0: Write the log buffer to the log file and flush the log file every second, but do nothing at transaction commit
# 1：the log buffer is written out to the log file at each transaction commit and the flush to disk operation is performed on the log file
# 2：the log buffer is written out to the file at each commit, but the flush to disk operation is not
innodb_flush_log_at_trx_commit = 1

# 设置InnoDB存储引擎的事务日志使用的缓冲区
innodb_log_buffer_size = 16M

# 设置InnoDB存放索引和表数据的最大缓冲区大小
innodb_buffer_pool_size = 1G
如果是独占服务器, 这个值可设为1/2内存大小

# 一个InnoDB事条日志的大小
innodb_log_file_size = 512M // 如果插入数据频率高, 可以设置的更大些
innodb_log_buffer_size = 128M // 倍数关系

# InnoDB最大并发线程数
innodb_thread_concurrency = 32
innodb_read_io_threads = 16
innodb_write_io_threads = 16
innodb_io_capactity = 10000
```

## 16G内存配置示例
```shell
[mysqld_safe]
nice 				= -15

[mysqld]
max_binlog_size 		= 256M	  #max size for binlog before rolling
expire_logs_days 		= 1	#binlog files older than this will be purged

## Per-Thread Buffers * (max_connections) = total per-thread mem usage
thread_stack 			= 256K	  #default: 32bit: 192K, 64bit: 256K
sort_buffer_size 		= 1M	  #default: 2M, larger may cause perf issues
read_buffer_size 		= 1M	  #default: 128K, change in increments of 4K
read_rnd_buffer_size 	        = 1M	  #default: 256K				
join_buffer_size 		= 1M	  #default: 128K
binlog_cache_size 		= 64K	  #default: 32K, size of buffer to hold TX queries

## Query Cache
query_cache_size 		= 32M     #global buffer
query_cache_limit 		= 512K	  #max query result size to put in cache

## Connections
max_connections 		= 2000	  #multiplier for memory usage via per-thread buffers
max_connect_errors 		= 100	  #default: 10
concurrent_insert		= 2	      #default: 1, 2: enable insert for all instances
connect_timeout			= 30	  #default -5.1.22: 5, +5.1.22: 10

## Table and TMP settings
max_heap_table_size 		= 1G	#recommend same size as tmp_table_size
bulk_insert_buffer_size 	= 1G	#recommend same size as tmp_table_size
tmp_table_size                  = 1G    #recommend 1G min

# set tmpdir to a ramdisk?
#tmpdir                         = /data/mysql-tmp0:/data/mysql-tmp1

## Thread settings
thread_concurrency		= 0  	#recommend 2x CPU cores [0 create as many as needed]
thread_cache_size		= 100 	#recommend 5% of max_connections

## MyISAM Engine
key_buffer			= 1M	#global buffer
myisam_sort_buffer_size		= 128M	#index buffer size for creating/altering indexes
myisam_max_sort_file_size	= 256M	#max file size for tmp table when creating/alering indexes

## InnoDB IO settings -  5.5.x and greater
innodb_write_io_threads 	= 16
innodb_read_io_threads		= 16

## InnoDB Plugin Independent Settings
innodb_file_per_table		#enable always
innodb_data_file_path		= ibdata1:128M;ibdata2:10M:autoextend
innodb_buffer_pool_size		= 12G 			#global buffer
innodb_additional_mem_pool_size	= 4M			#global buffer
innodb_flush_log_at_trx_commit	= 0				#2/0 = perf, 1 = ACID
innodb_log_file_size 		= 256M
innodb_log_buffer_size 		= 128M 			#global buffer
innodb_lock_wait_timeout 	= 300	
innodb_thread_concurrency	= 16			#recommend 2x core quantity
innodb_commit_concurrency	= 16			#recommend 4x num disks
innodb_flush_method		= O_DSYNC	    #O_DIRECT = local/DAS, O_DSYNC = SAN/iSCSI
innodb_support_xa		= 0		   		#recommend 0, disable xa to negate extra disk flush
skip-innodb-doublewrite
sync_binlog			= 0
```