## 表设计和存储引擎

常用的数据存储引擎包括MyISAM和InnoDB
* MyISAM单线程插入速度一流, 多线程插入会频繁触发表级锁, 性能下降明显
* InnoDB不能使用uuid作为主键、唯一索引、InnoDB中数据按照主键顺序存储，uuid随机主键在表数据量较大时候，会是性能梦魇。如

## 查询优化

### Large Offset

```
SELECT  t.*
FROM (
    SELECT  id
    FROM mytable
    ORDER BY id
    LIMIT 10000, 30
    ) q
JOIN mytable t
ON t.id = q.id
```

### SELECT-UPDATE
SELECT-UPDATE 场景主要指某业务逻辑中，需要先从数据库中选择符合特定条件的记录，然后对选择的记录进行更新。当多线程环境下，由于业务逻辑分为多步，如果使用多条SQL执行的话，会导致数据覆盖等逻辑不一致等问题。解决这个问题主要有以下几种方法：
1. SQL语句合并
   - 合并为1条SQL语句，对于简单逻辑有效，不容易修改调试，复杂逻辑无意义。
2. 使用事务
   - InnoDB支持行锁，InnoDB在读取行时可以加两种锁：读共享锁、写独占锁
   - 读共享锁：
      ```
      SELECT * FROM parent WHERE NAME = 'Jones' LOCK IN SHARE MODE;
      ```
      如果事务A获得了先获得了读共享锁，那么事务B之后仍然可以读取加了读共享锁的行数据，但必须等事务A commit或者roll back之后才可以更新或者删除加了读共享锁的行数据。 
   - 写独占锁： 
      ``` 
      SELECT counter_field FROM child_codes FOR UPDATE;
      UPDATE child_codes SET counter_field = counter_field + 1;
      ```
      如果事务A先获得了某行的写共享锁，那么事务B就必须等待事务A commit或者roll back之后才可以访问行数据。举几个例子：
      `select * from t for update` 会等待行锁释放之后，返回查询结果。
      `select * from t for update nowait` 不等待行锁释放，提示锁冲突，不返回结果
      `select * from t for update wait 5` 等待5秒，若行锁仍未释放，则提示锁冲突，不返回结果
      `select * from t for update skip locked` 查询返回查询结果，但忽略有行锁的记录
   - 这两种方式在事务(Transaction) 进行当中SELECT 到同一个数据表时，都必须等待其它事务数据被提交(Commit)后才会执行。而主要的不同在于LOCK IN SHARE MODE 在有一方事务要Update 同一个表单时很容易造成死锁。
   - 如果度操作响应要求高，使用写独占锁会阻塞读取。

## 插入更新优化
### INSERT INTO SELECT
```sql
INSERT INTO Table2(field1, field2, ...)
SELECT value1, value2, ... FROM Table1
```
要求目标表Table2必须存在，由于目标表Table2已经存在，所以我们除了插入源表Table1的字段外，还可以插入常量。

### 批量插入
修改 `my.ini/my.cnf` 配置文件，设置 `max_allowed_packet=128M`
```
INSERT INTO tableName(c1,c2,c3) VALUES(value1,valeu2,value3),(value11,value22,value33),(value111,value222,value333)
```

### 禁用索引提升插入效率
```
ALTER TABLE tblname DISABLE KEYS;
// loading the data
ALTER TABLE tblname ENABLE KEYS;
```
启用索引后会全表重建索引

### 数据插入更新
1. **INSERT IGNORE INTO**
   - 当插入数据时，如出现错误时，如重复数据，将不返回错误，只以警告形式返回。所以使用ignore请确保语句本身没有问题，否则也会被忽略掉。例如：
   ```
   INSERT IGNORE INTO books (name) VALUES ('MySQL Manual')
   ```
2. **ON DUPLICATE KEY UPDATE**
   - 当primary或者unique重复时，则执行update语句，如update后为无用语句，如id = id，则同1功能相同，但错误不会被忽略掉。例如，为了实现name重复的数据插入不报错，可使用一下语句：
   ```
   INSERT INTO books (name) VALUES ('MySQL Manual') ON duplicate KEY UPDATE id = id
   
   INSERT INTO authors (id, title)
   VALUES
   ("1", "1"), ("2", "2"), ("3", "3") agent_stats
   ON DUPLICATE KEY UPDATE title = VALUES(title)
   ```
3. **INSERT ... SELECT ... WHERE NOT EXIST**
   - 根据select的条件判断是否插入，可以不光通过primary和unique来判断，也可通过其它条件。例如：
   ```
   INSERT INTO books (name) SELECT 'MySQL Manual' FROM dual WHERE NOT EXISTS (SELECT id FROM books WHERE id = 1)
   ```
4. **REPLACE INTO**
   - 如果存在primary or unique相同的记录，则先删除掉。再插入新记录。
   ```
   REPLACE INTO books SELECT 1, 'MySQL Manual' FROM books
   ```
#### replace函数
- `replace(object, search, replace)`
- object代表字段，search代表要替换的字符（在字段中），replace代表替换的文本
   ```
   update `behavior_type_patterns` set `pattern`=replace(`pattern`,``,``);
   ```

#### 更新第一行
```
UPDATE tablename SET main_photo = 'yes' ORDER BY photo_id ASC LIMIT 1;
```

## JOIN
默认的 JOIN 是 INNER JOIN，INNER JOIN 和 LEFT JOIN 的区别是：
- INNER JOIN 只返回在两个表中都存在的记录。如果某个表中没有匹配的记录，该记录不会出现在结果集中。
- LEFT JOIN 返回左表（表A）的所有记录，以及右表（表B）中匹配的记录。如果右表中没有匹配的记录，结果集中会显示左表的记录，并且右表的字段会返回NULL。

```
SELECT something
FROM   master      parent
JOIN   master      child ON child.parent_id = parent.id
LEFT   JOIN second parentdata ON parentdata.id = parent.secondary_id
LEFT   JOIN second childdata ON childdata.id = child.secondary_id
WHERE  parent.parent_id = 'rootID'
```

## 索引

使用传统MyISAM引擎更新索引，主要进行以下三个步骤：
- 将原是表数据拷贝到一个新表中（需要占用大量磁盘IO）
- 重建所有索引（根据数据量而定）
- 将临时表重新命名（会非常快）

使用InnoDB引擎时，可以设置 `ALGORITHM=INPLACE` 可以省掉拷贝数据的步骤

参考：
* http://dev.mysql.com/doc/refman/5.6/en/alter-table.html

### 手动重建索引
不使用Alter语句，手动重建MyISAM表索引，可以省50%左右的时间，具体代码如下：
```
CREATE TABLE phpbb_posts_new LIKE phpbb_posts; // 创建新表
ALTER TABLE phpbb_posts_new DROP INDEX post_text; // 在新表中删除索引
ALTER TABLE phpbb_posts_new DISABLE KEYS; // 停用Keys
INSERT INTO phpbb_posts_new SELECT * FROM phpbb_posts; // 将旧表中的数据插入到新表
ALTER TABLE phpbb_posts_new ENABLE KEYS; // 启用新表的Keys
ALTER TABLE phpbb_posts RENAME phpbb_posts_old; // 将旧表重命名
ALTER TABLE phpbb_posts_new RENAME phpbb_posts; // 将新表重命名
DROP TABLE phpbb_posts_old; // 删除旧表
```
当数据量在百万量级，表文件小于1GB时，此方法较为适合。数据量过大，如表文件>100GB，条目超过5M，在原有表上添加修改索引将会是异常麻烦的事情，所以尽量避免。

## 存储过程/事件
```
# 开启事件
SET GLOBAL event_scheduler = ON;

# 选择
SET @stmt_text=CONCAT("SELECT @score = SUM(`score`), @maxscore=SUM(`maxscore`) FROM ", answertable, "WHERE `idParticipation`= ",  partid);
PREPARE stmt FROM @stmt_text;
EXECUTE stmt USING @a;

# 更新
SET @stmt_text=CONCAT("UPDATE", participationtable, " SET `score`=@score, `maxscore`=@maxscore WHERE `idParticipation`=", partid);
PREPARE stmt FROM @stmt_text;
EXECUTE stmt USING @a;
DEALLOCATE PREPARE stmt;
```

## 函数
```
# 判断是否为null
ifnull(null, 0)

# IF
IF(cond, true expr, false expr)

# binary转String
SELECT binary(user_email) FROM user WHERE binary(user_name) = 'Foxtail3000'
```

### 时间日期
`now() | sysdate() | curdate() | curtime() | utc_timestamp() | utc_date() | utc_time() `

```
set @dt = '2008-09-10 07:15:30.123456';

select date(@dt);        -- 2008-09-10
select time(@dt);        -- 07:15:30.123456
select year(@dt);        -- 2008
select quarter(@dt);     -- 3
select month(@dt);       -- 9
select week(@dt);        -- 36
select day(@dt);         -- 10
select hour(@dt);        -- 7
select minute(@dt);      -- 15
select second(@dt);      -- 30
select microsecond(@dt); -- 123456

# date_add(date1,date2)
select date_add(@dt, interval 1 day);        -- add 1 day
select date_add(@dt, interval 1 hour);       -- add 1 hour
select date_add(@dt, interval 1 minute);     -- ...
select date_add(@dt, interval 1 second);
select date_add(@dt, interval 1 microsecond);
select date_add(@dt, interval 1 week);
select date_add(@dt, interval 1 month);
select date_add(@dt, interval 1 quarter);
select date_add(@dt, interval 1 year);
select date_add(@dt, interval -1 day);       -- sub 1 day
select date_add(@dt, interval '01:15:30' hour_second);

# datediff(date1,date2)
select datediff('2008-08-08', '2008-08-01');  -- 7
select datediff('2008-08-01', '2008-08-08');  -- -7

# timediff(time1,time2)
select timediff('2008-08-08 08:08:08', '2008-08-08 00:00:00'); -- 08:08:08
select timediff('08:08:08', '00:00:00');                       -- 08:08:08

# time_to_sec(time), sec_to_time(seconds) 时间、秒转换函数
select time_to_sec('01:00:05');  -- 3605
select sec_to_time(3605);        -- '01:00:05'

# to_days(date), from_days(days) 日期、天数转换函数
select to_days('0000-00-00');  -- 0
select to_days('2008-08-08');  -- 733627
select from_days(0);           -- '0000-00-00'
select from_days(733627);      -- '2008-08-08'

# str_to_date(str, format) 字符串转换为日期
select str_to_date('08/09/2008', '%m/%d/%Y');                   -- 2008-08-09
select str_to_date('08/09/08'  , '%m/%d/%y');                   -- 2008-08-09
select str_to_date('08.09.2008', '%m.%d.%Y');                   -- 2008-08-09
select str_to_date('08:09:30', '%h:%i:%s');                     -- 08:09:30
select str_to_date('08.09.2008 08:09:30', '%m.%d.%Y %h:%i:%s'); -- 2008-08-09 08:09:30

# unix_timestamp() / from_unixtime()
select unix_timestamp();                       -- 1218290027
select unix_timestamp('2008-08-08');           -- 1218124800
select unix_timestamp('2008-08-08 12:30:00');  -- 1218169800
select from_unixtime(1218290027);              -- '2008-08-09 21:53:47'
select from_unixtime(1218124800);              -- '2008-08-08 00:00:00'
select from_unixtime(1218169800);              -- '2008-08-08 12:30:00'
select from_unixtime(1218169800, '%Y %D %M %h:%i:%s %x'); -- '2008 8th August 12:30:00 2008'

# 时间戳（timestamp）转换
timestamp(date)                                     -- date to timestamp
timestamp(dt,time)                                  -- dt + time
select timestamp('2008-08-08');                         -- 2008-08-08 00:00:00
select timestamp('2008-08-08 08:00:00', '01:01:01');    -- 2008-08-08 09:01:01
select timestamp('2008-08-08 08:00:00', '10 01:01:01'); -- 2008-08-18 09:01:01

# timestampadd(unit,interval,datetime_expr)
select timestampadd(day, 1, '2008-08-08 08:00:00');     -- 2008-08-09 08:00:00
select date_add('2008-08-08 08:00:00', interval 1 day); -- 2008-08-09 08:00:00 方法比较

# timestampdiff(unit,datetime_expr1,datetime_expr2)
select timestampdiff(year,'2002-05-01','2001-01-01');                    -- -1
select timestampdiff(day ,'2002-05-01','2001-01-01');                    -- -485
select timestampdiff(hour,'2008-08-08 12:00:00','2008-08-08 00:00:00');  -- -12
select datediff('2008-08-08 12:00:00', '2008-08-01 00:00:00');           -- 7  方法比较
```

## 加锁分析

和其他的并发问题一样，MySQL也是通过加锁进行并发控制的。锁就会涉及读写锁、锁粒度等问题。为了防止一个线程访问数据时，其他数据修改了同一部分数据，一种办法是加一把大锁，这样固然能解决数据安全问题，但是带来的影响是不支持并发。所以一种经典解决办法是定义两种类型的锁，共享锁(shared lock)和排它锁(exclusive lock)，也叫读锁(read lock)和写锁(write lock)。读锁是共享的，写锁是排他的。多个读锁可以同时存在，但是写锁不能和读锁和写锁共存。在读多写少的场景比较适合读写锁。
在锁的大小范围上，可以定义锁的粒度,我们可以锁全部数据，也可以只锁我们访问的数据，锁的粒度越小，所需资源越多，增加的系统开销也越大。在数据库概念中分为了表级锁和行级锁。

MySQL的锁机制比较简单，其最显著的特点是不同的存储引擎支持不同的锁机制。
* MyISAM和MEMORY存储引擎采用的是表级锁（table-level locking）
* InnoDB存储引擎既支持行级锁（row-level locking），也支持表级锁，但默认情况下是采用行级锁

不同锁的性能归纳如下：
* 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低
* 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高

### MyISAM表级锁
MyISAM存储引擎只支持表锁，可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺：
```
show status like 'table%';
```
如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。

MySQL的表级锁有两种模式：表共享读锁（Table Read Lock）和表独占写锁（Table Write Lock），**对MyISAM表的读操作，不会阻塞其他用户对同一表的读请求，但会阻塞对同一表的写请求；对 MyISAM表的写操作，则会阻塞其他用户对同一表的读和写操作**；MyISAM表的读操作与写操作之间，以及写操作之间是串行的。当一个线程获得对一个表的写锁后，只有持有锁的线程可以对表进行更新操作。其他线程的读、写操作都会等待，直到锁被释放为止。

MyISAM在执行查询语句（SELECT）前，会自动给涉及的所有表加读锁，在执行更新操作 （UPDATE、DELETE、INSERT等）前，会自动给涉及的表加写锁，这个过程并不需要用户干预，因此，一般不需要直接用LOCK TABLE命令给MyISAM表显式加锁。

给MyISAM表显示加锁，一般是为了在一定程度模拟事务操作，实现对某一时间点多个表的一致性读取。
```
Lock tables orders read local, order_detail read local;  
Select sum(total) from orders;  
Select sum(subtotal) from order_detail;  
Unlock tables;
```
MyISAM总是一次获得SQL语句所需要的全部锁。这也正是MyISAM表不会出现死锁（Deadlock Free）的原因。

一个进程请求某个 MyISAM表的读锁，同时另一个进程也请求同一表的写锁，写进程先获得锁。不仅如此，即使读请求先到锁等待队列，写请求后到，写锁也会插到读锁请求之前。

MySQL认为写请求一般比读请求要重要。这也正是MyISAM表不太适合于有大量更新操作和查询操作应用的原因，大量的更新操作会造成查询操作很难获得读锁，从而可能永远阻塞。

### 事务

事务是一组原子性的SQL查询，或者说一个独立的工作单元。事务内的语句，要么全部执行成功，要么全部执行失败。常见的例子是银行转帐，A转帐给B，包括三个操作，检查A的余额大于100元，从A的账户减100元，给B的账户加100元。事务的四个特性是ACID。
* 原子性(Atomicity): 一个事务必须被视为一个不可分割的最小工作单元，事务中的操作要么全部执行成功，要么全部失败会滚，不可能只执行其中的一部分。
* 一致性(consistency) 数据库总是要从一个一致的状态转换到另一个一致的状态。需要确保当从A减去100时若系统崩溃，账户不会损失100元。
* 隔离性(isolation)通常来说，一个事务在最终提交前对其他事务是不可见的。为满足不同的并发场景，定义了不同的隔离级别，从低到高是读未提交，读已提交，可重复读和序列化。序列化能够保证最强的一致性，其他隔离级别的一致性依次将此，并发性更好。读未提交会看到其他事务没有提交的操作，称为脏读。读已提交只能看到其他事务已经提交的修改，但是不能重复读，因为一个事务内两次读看到的结果不同。可重复读解决了不可重复读的问题，但是有幻读的问题，就是在读取某个范围的记录时，其他事务在该范围内插入了新的数据，两次读的结果不一致。InnoDB通过多版本并发控制(MVCC)解决了幻读的问题，可重复读是MySQL的默认隔离级别。序列化是最高的隔离级别，会让事务串行执行，在读取的每一行数据加锁，会造成大量的锁征用。
* 持久性(durability) 持久性要保证提交了事务后，即使系统崩溃，修改的数据也不会丢失。持久性也分为不同的级别，通过备份等手段可以提高持久性。

MySQL中的事务
- MySQL默认使用自动提交，可以通过AUTOCOMMIT变量来启用或禁止自动提交，设置AUTOCOMMIT为0后，可以通过COMMIT提交事务或通过ROLLBACK回滚事务。

### 死锁
死锁是指多个事务循环等待已持有的资源。如A获取了资源1要获取资源2，同时B获取了资源2要去获取资源1.这样就形成了死锁。死锁都发生在多个资源的加载顺序不一致导致，MySQL中的死锁多发生在事务中的加锁引起。MySQL中提供了死锁检测和回滚事务中的一个事务打破死锁的功能。

## EXPLAIN
```
explain select surname,first_name form a,b where a.id=b.id
```
EXPLAIN显示了mysql如何使用索引来处理select语句以及连接表。可以帮助选择更好的索引和写出更优化的查询语句。
- table	显示这一行的数据是关于哪张表的
- type	这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、eq_reg、ref、range、indexhe和ALL
- possible_keys	显示可能应用在这张表中的索引。如果为空，没有可能的索引。可以为相关的域从WHERE语句中选择一个合适的语句
- key	实际使用的索引。如果为NULL，则没有使用索引。很少的情况下，MYSQL会选择优化不足的索引。这种情况下，可以在SELECT语句中使用USE INDEX（indexname）来强制使用一个索引或者用IGNORE INDEX（indexname）来强制MYSQL忽略索引
- key_len	使用的索引的长度。在不损失精确性的情况下，长度越短越好
- ref	显示索引的哪一列被使用了，如果可能的话，是一个常数
- rows	MYSQL认为必须检查的用来返回请求数据的行数
- Extra	关于MYSQL如何解析查询的额外信息。但这里可以看到的坏的例子是Using temporary和Using filesort，意思MYSQL根本不能使用索引，结果是检索会很慢
- Distinct	一旦MYSQL找到了与行相联合匹配的行，就不再搜索了
   - Not exists	MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了
   - Range checked for each Record（index map:#）	没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一
   - Using filesort	看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行
   - Using index	列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候
   - Using temporary	看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上
   - Where used	使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题不同连接类型的解释（按照效率高低的顺序排序）
   - system	表只有一行：system表。这是const连接类型的特殊情况
   - const	表中的一个记录的最大值能够匹配这个查询（索引可以是主键或惟一索引）。因为只有一行，这个值实际就是常数，因为MYSQL先读这个值然后把它当做常数来对待
   - eq_ref	在连接中，MYSQL在查询时，从前面的表中，对每一个记录的联合都从表中读取一个记录，它在查询使用了索引为主键或惟一键的全部时使用
   - ref	这个连接类型只有在查询使用了不是惟一或主键的键或者是这些类型的部分（比如，利用最左边前缀）时发生。对于之前的表的每一个行联合，全部记录都将从表中读出。这个类型严重依赖于根据索引匹配的记录多少—越少越好
   - range	这个连接类型使用索引返回一个范围中的行，比如使用>或<查找东西时发生的情况
   - index	这个连接类型对前面的表中的每一个记录联合进行完全扫描（比ALL更好，因为索引一般小于表数据）
   - ALL	这个连接类型对于前面的每一个记录联合进行完全扫描，这一般比较糟糕，应该尽量避免

## 表分区
### 手动分区

```
ALTER TABLE n_nv_rela PARTITION BY RANGE(TO_DAYS(`pubdate`))
(
	PARTITION p20130101 VALUES LESS THAN(TO_DAYS('2013-01-02 00:00:00')),
	PARTITION p20130102 VALUES LESS THAN(TO_DAYS('2013-01-03 00:00:00')),
	PARTITION p20130103 VALUES LESS THAN(TO_DAYS('2013-01-04 00:00:00')),
	PARTITION p20130104 VALUES LESS THAN(TO_DAYS('2013-01-05 00:00:00')),
	PARTITION p20130105 VALUES LESS THAN(TO_DAYS('2013-01-06 00:00:00')),
	PARTITION p0 VALUES LESS THAN(MAXVALUE)
)
```

### 更改分区

```
ALTER TABLE n_nv_rela REORGANIZE PARTITION p0 INTO
(
    PARTITION p20130106 VALUES LESS THAN(TO_DAYS('2013-01-07 00:00:00')),
    PARTITION p20130107 VALUES LESS THAN(TO_DAYS('2013-01-08 00:00:00')),
    PARTITION p20130108 VALUES LESS THAN(TO_DAYS('2013-01-09 00:00:00')),
    PARTITION p0 VALUES LESS THAN(MAXVALUE)
)
```

### 自动表分区脚本

目标表必须已经创建好分区
```
DELIMITER // 
DROP PROCEDURE IF EXISTS create_Partition
CREATE PROCEDURE `create_Partition`
(
    IN `databaseName` VARCHAR(50), 
    IN `tableName` VARCHAR(50), 
    IN `runDate` DATE, 
    IN `gap` INT
)
L_END:BEGIN

    DECLARE last_partition VARCHAR(255);
    DECLARE last_partition_date Date;
    
    DECLARE P_NAME VARCHAR(255);
    DECLARE P_DESCRIPTION Date;
    
    DECLARE i INT DEFAULT 1;
    
    SELECT PARTITION_NAME
    INTO last_partition
    FROM information_schema.PARTITIONS 
    WHERE TABLE_SCHEMA = databaseName 
        AND TABLE_NAME = tableName
        AND PARTITION_NAME != "p0"
    ORDER BY partition_description DESC 
    LIMIT 1;
    
    IF last_partition <=> "" THEN
        SELECT "Partition table not is exist" AS "*****ERROR*****";
        LEAVE L_END;
    END IF;
    
    SET last_partition = REPLACE(last_partition, 'p', '');
    SET last_partition_date = str_to_date(last_partition,'%Y%m%d');
    SET @S=CONCAT('ALTER TABLE ',tableName,' REORGANIZE PARTITION p0 INTO (');
            
    WHILE gap * (i+1) <= datediff(runDate,last_partition_date) DO
    
        SET P_NAME = REPLACE(adddate(last_partition_date, INTERVAL i*gap day), '-', '');
        SET P_DESCRIPTION = adddate(last_partition_date, INTERVAL (i+1)*gap day);
        
        SET @S=CONCAT(@S, 'PARTITION p',P_NAME,' VALUES LESS THAN (TO_DAYS(\'',P_DESCRIPTION,'\')),');
        
        SET i = i + 1;
    
    END WHILE;
    
    SET @S=CONCAT(@S, 'PARTITION p0 VALUES LESS THAN(MAXVALUE))');
    
    -- Insert test values (@S);
    
    SELECT @S;
    PREPARE stmt2 FROM @S;
    EXECUTE stmt2;
    DEALLOCATE PREPARE stmt2;

END L_END
```

### 调用事件

```
CREATE EVENT auto_set_partitions  
    ON SCHEDULE  
    EVERY 15 DAY   
DO  
BEGIN  
	CALL create_Partition('database_name','table_name');  
    /* 如果需要向多个表分区,可以写多个 CALL 调用    
    CALL create_Partition('database_name','table_name');  
    */  
END
```

## 运行状态

大部分状态对应很快的操作，只要有一个线程保持同一个状态好几秒钟，那么可能是有问题发生了，需要检查一下。
- Checking table
   - 正在检查数据表（这是自动的）。
- Closing tables
    - 正在将表中修改的数据刷新到磁盘中，同时正在关闭已经用完的表。这是一个很快的操作，如果不是这样的话，就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中。
- Connect Out
    - 复制从服务器正在连接主服务器。
- Copying to tmp table on disk
    - 由于临时结果集大于 tmp_table_size，正在将临时表从内存存储转为磁盘存储以此节省内存。
- Creating tmp table
    - 正在创建临时表以存放部分查询结果。
- deleting from main table
    - 服务器正在执行多表删除中的第一部分，刚删除第一个表。
- deleting from reference tables
    - 服务器正在执行多表删除中的第二部分，正在删除其他表的记录。
- Flushing tables
    - 正在执行 FLUSH TABLES，等待其他线程关闭数据表。
- Killed
    - 发送了一个kill请求给某线程，那么这个线程将会检查kill标志位，同时会放弃下一个kill请求。MySQL会在每次的主循环中检查kill标志位，不过有些情况下该线程可能会过一小段才能死掉。如果该线程程被其他线程锁住了，那么kill请求会在锁释放时马上生效。
- Locked
    - 被其他查询锁住了。
- Sending data
    - 正在处理 SELECT 查询的记录，同时正在把结果发送给客户端。
    - 从server端发送数据到客户端，也有可能是接收存储引擎层返回的数据，再发送给客户端，数据量很大时尤其经常能看见备注：Sending Data不是网络发送，是从硬盘读取，发送到网络是Writing to net
    - 建议：通过索引或加上LIMIT，减少需要扫描并且发送给客户端的数据量
- Sorting for group
    - 正在为 GROUP BY 做排序。
- Sorting for order
    - 正在为 ORDER BY 做排序。
- Opening tables
    - 这个过程应该会很快，除非受到其他因素的干扰。例如，在执 ALTER TABLE 或 LOCK TABLE 语句行完以前，数据表无法被其他线程打开。正尝试打开一个表。
- Removing duplicates
    - 正在执行一个 SELECT DISTINCT 方式的查询，但是MySQL无法在前一个阶段优化掉那些重复的记录。因此，MySQL需要再次去掉重复的记录，然后再把结果发送给客户端。
- Reopen table
    - 获得了对一个表的锁，但是必须在表结构修改之后才能获得这个锁。已经释放锁，关闭数据表，正尝试重新打开数据表。
- Repair by sorting
    - 修复指令正在排序以创建索引。
- Repair with keycache
    - 修复指令正在利用索引缓存一个一个地创建新索引。它会比 Repair by sorting 慢些。
- Searching rows for update
    - 正在讲符合条件的记录找出来以备更新。它必须在 UPDATE 要修改相关的记录之前就完成了。
- Sleeping
    - 正在等待客户端发送新请求.
- System lock
    - 正在等待取得一个外部的系统锁。如果当前没有运行多个 mysqld 服务器同时请求同一个表，那么可以通过增加 –skip-external-locking参数来禁止外部系统锁。
- Upgrading lock
    - INSERT DELAYED 正在尝试取得一个锁表以插入新记录。
- Updating
    - 正在搜索匹配的记录，并且修改它们。
- User Lock
    - 正在等待 GET_LOCK()。
- Waiting for tables
    - 该线程得到通知，数据表结构已经被修改了，需要重新打开数据表以取得新的结构。然后，为了能的重新打开数据表，必须等到所有其他线程关闭这个表。以下几种情况下会产生这个通知：FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE, 或 OPTIMIZE TABLE。
- waiting for handler insert
    - INSERT DELAYED 已经处理完了所有待处理的插入操作，正在等待新的请求。
-  statistics
    - 进行数据统计以便解析执行计划，如果状态比较经常出现，有可能是磁盘IO性能很差建议：查看当前io性能状态，例如iowait

## 运维优化

### 探明瓶颈

* 如果程序设计有数据库操作逻辑层，可以从这个层的统计数据看到每个请求花费的时间，如果'''平均时间'''已经不能让你容忍的话，数据库已经是瓶颈了。
* 在数据库的服务器上使用top命令，看看mysql服务器占用资源的情况，看看机子的'''平均负载'''。如果服务器的平均负载已经很高，mysql占用了块100%的cpu资源，说明mysql服务器很忙了。
* 在数据库服务器上使用iostat命令，看看'''磁盘IO'''，如果block住的操作比较多的话，说明数据库操作还是过于频繁了，磁盘都响应不急了。
* 建议打开'''mysql慢查询'''日志，这样grep select看一下日志中的慢查询的数量，如果数量较多，说明慢查询的数量很多，需要进行调整了。

### 优化步骤

* **数据库大**：考虑**分库**和**分表**，扩充容量，实现负载均衡
* **查询慢**：检查一下我们的sql查询语句，利用mysql的explain语句看看是不是使用了索引，如果没有使用索引，那我们需要在相应的字段上建上索引，反复的使用explain，寻找到一个合适的索引。
   - 索引越少越好
   - 最窄的字段放在键的左边
   - 尽量避免排序、临时表和表扫描
* **更新慢**：主从热备来分担读写的压力，写操作放到mysql的主服务器上，在备份的数据库服务器上进行读操作，由于可以有多个热备mysql，于是可以将读操作分布在多个热备上面，从而将读操作均衡开来，提高读操作的性能。
* **缓存**
   - 增加key cache，thread cache、查询cache
   - 应用程序层增加一个memcached这样的通用cache
   - 对于少量数据，但是操作频繁的表使用mysql提供的内存heap表

### IO层面
* 根据IO_UTIL进行参数优化

### 规划层面

* 对数据库的操作建议**分层处理**，一层是**数据库操作逻辑层**，一层是**数据库cache层**，可以很方便在未来对数据库进行划分部署、分库分表扩展。
* 增加**监控**：监控mysql的慢查询日志，监控mysql的请求情况。

## 异常汇总
### InnoDB: Error: io_setup() failed with EAGAIN
```shell
140505 16:05:59  InnoDB: Warning: io_setup() failed with EAGAIN. Will make 5 attempts before giving up.
InnoDB: Warning: io_setup() attempt 1 failed.
InnoDB: Warning: io_setup() attempt 2 failed.
InnoDB: Warning: io_setup() attempt 3 failed.
InnoDB: Warning: io_setup() attempt 4 failed.
InnoDB: Warning: io_setup() attempt 5 failed.
140505 16:06:02  InnoDB: Error: io_setup() failed with EAGAIN after 5 attempts.
```
解决方法：
- 错误代码EAGAIN表明超出了可用event限制的最大值：
   ```
   cat /proc/sys/fs/aio-max-nr
   65536
   ```
- vi /etc/sysctl.conf
   - 增加 `fs.aio-max-nr=262144`
- 更新状态
   ```
   sysctl -p
   /etc/init.d/mysql start
   ```

### [Warning] Can't create test file 

- 这个问题主要原因是在 Ubuntu 服务器上更改了 Mysql 的 datadir
- 解决方法，设置apparmor，/etc/apparmor.d/usr.sbin.mysqld，规定了mysql使用的数据文件路径权限，在这个文件中参考已有内容添加新datadir对应的配置项。

### Disable ONLY_FULL_GROUP_BY

`vi /etc/mysql/my.cnf`
```
[mysqld]  
sql_mode = "STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
```
重启系统

### com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
```
The last packet successfully received from the server was 18,632 milliseconds ago.  The last packet sent successfully to the server was 259 milliseconds ago.
...
```
最简单的解决办法是增加 wait_timeout 配置项，`wait_timeout = 31536000`

## 其他

### 忘记管理员密码
```shell
mysqld --skip-grant-tables
```

### 开启日志
```shell
# 查询日志功能是否开启
# 可以通过general_log_file 找到日志文件位置
SHOW VARIABLES LIKE 'general%';

# 开启日志
set GLOBAL general_log='ON';
```