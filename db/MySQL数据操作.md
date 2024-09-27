## 数据同步
### Master 设置

创建一个专门用于同步的账户
```
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO repl@'192.168.0.%' IDENTIFIED BY '<密码>';
```

my.ini/cnf设置：
```
server-id = 106
log-bin=/var/lib/mysql/binlog
expire_logs_days = 1 // 可以根据设长一点
binlog-do-db = news // 需要同步的表
binlog-do-db = sae
binlog-do-db = market_data
```
重启MySQL，登录mysql获取log文件名和当前记录位置：
```
mysql> show master status;
```
记录 '''File''' '''Position'''

### Slave 设置

my.ini/cnf设置
```
server-id = 107
```
重启mysql，并登录
```
mysql> stop slave;
mysql> change master to master_host='[host]',master_user='[用户名]',master_password='[密码]',master_log_file='[日志文件名]',master_log_pos=[地址];
// mysql> CHANGE MASTER TO MASTER_HOST='192.168.220.123',MASTER_USER='repl', MASTER_PASSWORD='<密码>',MASTER_LOG_FILE='binlog.000001',MASTER_LOG_POS=154;
mysql> start slave;
mysql> show slave status\G
```

在主库插入一条数据，看从库是否有更新。

### 清理MySQL binlog日志

自动清理，配置my.cnf：
```
expire_logs_days = 10
```
在运行时修改：
```
mysql> show binary logs;
mysql> show variables like '%log%';
mysql> set global expire_logs_days = 10;
```
手动删除10天前的MySQL binlog日志：
```
mysql> PURGE MASTER LOGS BEFORE DATE_SUB(CURRENT_DATE, INTERVAL 10 DAY);
mysql> show master logs;
```

### 其他相关命令
```
# 查看所有连接到Master的Slave信息
mysql> show slave hosts;

# 查看Master状态信息
mysql> show master status;

# 查看Slave状态信息
mysql> show slave status;

# 查看所有二进制日志
mysql> show binary logs;

# 查看二进制日志中的事件
mysql> show binlog events [IN log_file];
```

## 数据迁移
### 导出
#### mysqldump
```shell
# 一般形式
mysqldump -u root -p news articles > articles_history.sql
mysqldump -h [host] -P[port_number] -u [username] -p [db_name] [table_name] > [sql_file]

# 条件导出语句
mysqldump.exe -u root -p sae W_TR_a --where "datetime >= '2013-09-01 00:00:00' and datetime < '2013-09-08 00:00:00' and char_length(w) < 7" > /data/backup/a.sql

# 全库导出
mysqldump -u$USER -p$PASSWD -h127.0.0.1 -P3306 --routines --default-character-set=utf8 --lock-all-tables --add-drop-database -A > db.all.sql

# 指定库导出
mysqldump -u$USER -p$PASSWD -h127.0.0.1 -P3306 --routines --default-character-set=utf8 --databases mysql > db.sql

# 指定表导出
mysqldump -u$USER -p$PASSWD -h127.0.0.1 -P3306 --routines --default-character-set=utf8 --tables mysql user > db.table.sql
```

导出TXT
- 导出某个查询sql的数据为txt格式文件到本地的目录(各数据值之间用"制表符"分隔)
- 例如sql为 `select user,host,password from mysql.user;`
   ```
   mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 --default-character-set=utf8 --skip-column-names -B -e 'select user,host,password from mysql.user;' > mysql_user.txt
   ```
导出逗号分割文件
* 导出某个查询sql的数据为txt格式文件到MySQL服务器
* 登录MySQL，将默认的制表符换成逗号(适应csv格式文件)
* 指定的路径，mysql要有写的权限，最好用tmp目录，文件用完之后删除
   ```
   SELECT user,host,password FROM mysql.user INTO OUTFILE '/tmp/mysql_user.csv' FIELDS TERMINATED BY ',';
   ```

参数：
* -A 全库备份
* --routines 备份存储过程和函数
* --default-character-set=utf8 设置字符集
* --lock-all-tables 全局一致性锁
* --add-drop-database 在每次执行建表语句之前，先执行DROP TABLE IF EXIST语句
* --no-create-db 不输出CREATE DATABASE语句
* --no-create-info 不输出CREATE TABLE语句
* --databases 将后面的参数都解析为库名
* --tables 第一个参数为库名 后续为表名

### 导入

#### 全库导入

恢复全库数据到MySQL，因为包含mysql库的权限表，导入完成需要执行`FLUSH PRIVILEGES;`
```shell
mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 --default-character-set=utf8 < db.all.sql
```
或登录MySQL，执行source命令，后面的文件名要用绝对路径
```shell
mysql> source /tmp/db.all.sql;
```

参数：
* --skip-column-names 不显示数据列的名字
* -B 以批处理的方式运行mysql程序.查询结果将显示为制表符间隔格式.
* -e 执行命令后,退出

#### 恢复某个库
```shell
mysql -u root -p news < C:\news_1310101709.sql
mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 --default-character-set=utf8 mysql < db.table.sql
```
或登录MySQL，执行source命令，后面的文件名要用绝对路径
```shell
mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 --default-character-set=utf8
mysql> use mysql;
mysql> source /tmp/db.table.sql;
```
忽略错误导入
```shell
mysql -u userName -p -f -D dbName < script.sql
```

#### 从TXT文件恢复表

恢复MySQL服务器上面的txt格式文件(需要FILE权限,各数据值之间用"制表符"分隔)
```shell
mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 --default-character-set=utf8
mysql> use mysql;
mysql> LOAD DATA INFILE '/tmp/mysql_user.txt' INTO TABLE user;
```

#### 从逗号分割文件恢复表

恢复MySQL服务器上面的csv格式文件(需要FILE权限,各数据值之间用"逗号"分隔)
```shell
mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 --default-character-set=utf8
mysql> use mysql;
mysql> LOAD DATA INFILE '/tmp/mysql_user.csv' INTO TABLE user FIELDS TERMINATED BY ',';
```

### 数据目录迁移
准备工作
* 结束一切依赖MySQL的进程
* 关闭MySQL
   ```
   service mysql stop
   ```

拷贝数据文件
```shell
sudo cp -R -p /var/lib/mysql /path/to/new/mysqldb
```

修改文件夹权限
```shell
sudo chown -R mysql:mysql /path/to/new/mysqldb
```

修改MySQL配置文件 vi /etc/mysql/my.cnf
```shell
# datadir=/var/lib/mysql
datadir=/path/to/new/mysqldb
```

修改apparmor配置，否则会因为没有写权限而无法启动，`vi /etc/apparmor.d/usr.sbin.mysqld`
```shell
# 在/usr/sbin/mysqld配置项中添加
/path/to/new/mysqldb r,
/path/to/new/mysqldb/** rwk,
/path/to/new/mysqldb/mysqld.pid rw,
/path/to/new/mysqldb/mysqld.sock w,
```
保存并退出，reload apparmor配置
```shell
/etc/init.d/apparmor reload
```

启动MySQL
```shell
service start mysql
```

## 流式读取和批量插入

### 流式读取
```
Connection conn = PooledDataSource.getDataSource("musical_sharing").getConnection();
Statement stmt = conn.createStatement(ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY);
stmt.setFetchSize(Integer.MIN_VALUE);
ResultSet rs = stmt.executeQuery(sql);
while (rs.next()) {
        ...
}
```
### 批量插入
jdbc连接字串设置
```shell
rewriteBatchedStatements=true
```
示例代码
```
int batchSize = 1000;
PreparedStatement ps = connection.prepareStatement("insert into tb1 (c1,c2,c3...) values (?,?,?...)");

for (int i = 0; i < list.size(); i++) {

    ps.setXXX(list.get(i).getC1());
    ps.setYYY(list.get(i).getC2());
    ps.setZZZ(list.get(i).getC3());

    ps.addBatch();

    if ((i + 1) % batchSize == 0) {
        ps.executeBatch();
    }
}

if (list.size() % batchSize != 0) {
    ps.executeBatch();
}
```

## 常用SQL语句

### 文本替换

```shell
UPDATE next_url_reg_rules[表名]
SET next_url_reg_rules[表名].next_url_template[字段名] = replace(next_url_reg_rules.next_url_template, "{", "{{");

UPDATE next_url_reg_rules
SET next_url_reg_rules.next_url_template = replace(next_url_reg_rules.next_url_template, "{", "{{");
```

### 自增长清零
```shell
ALTER TABLE `表` AUTO_INCREMENT=1;
```

### 改变特定字段的编码
存储uuid时用binary或者ascii编码
```shell
ALTER TABLE N_V_r modify column a_id char(36) character set ascii not null;
```

### 时间相关汇总统计
#### 以天为单位 N_TR
```
SELECT
	n_tr.n,
	86400*FLOOR(UNIX_TIMESTAMP(n_tr.pubdate)/86400),
	sum(n_tr.count),
	sum(n_tr.trend),
	sum(n_tr.favor)
FROM n_tr, stock_names 
WHERE 
	n_tr.n = stock_names.name 
	AND pubdate >= '2013-01-01 00:00:00' 
	AND pubdate < '2013-01-02 00:00:00'
GROUP BY FLOOR(UNIX_TIMESTAMP(pubdate)/86400);
```

#### 以天为单位 主题
```
SELECT 
	3600000*ROUND(UNIX_TIMESTAMP(datetime)/3600), 
	sum(count),sum(trend)+sum(favor)+0.8*sum(dev)+0.8*sum(pop)+0.5*sum(ctrl)-0.8*sum(risk)-0.2*sum(trade), 
	sum(trend), 
	sum(favor) 
FROM W_TR_a_temp 
WHERE 
	w = "美元" 
	and datetime >= "2013-08-01 00:00:00" 
	and datetime < "2013-08-30 00:00:00" 
GROUP BY ROUND(UNIX_TIMESTAMP(datetime)/3600)
```

模糊查询
```
SELECT 
	3600000*ROUND(UNIX_TIMESTAMP(datetime)/3600), 
	sum(count),sum(trend)+sum(favor)+0.8*sum(dev)+0.8*sum(pop)+0.5*sum(ctrl)-0.8*sum(risk)-0.2*sum(trade), 
	sum(trend), 
	sum(favor) 
FROM W_TR_a_temp 
WHERE 
	w IN (SELECT w FROM w_temp WHERE w like "%美元%")
	and datetime >= "2013-08-01 00:00:00" 
	and datetime < "2013-08-30 00:00:00" 
GROUP BY ROUND(UNIX_TIMESTAMP(datetime)/3600);
```

#### 一般形式 行情

```
select 
	86400000*FLOOR(UNIX_TIMESTAMP(datetime)/86400),
	price 
from stock_prices 
where 
	stock_prices.code = "600036" 
	and datetime >= "2013-05-28 00:00:00" 
	and datetime < "2014-02-24 00:00:00" 
GROUP BY FLOOR(UNIX_TIMESTAMP(datetime)/86400)
```

#### 行情修正
```
select 
	stock_prices.datetime,
	stock_prices.price,
	t.m,
	t.a 
from stock_prices, (SELECT * FROM stock_price_adjustments
WHERE code = "600036" 
	and datetime >= "2013-05-28 00:00:00" 
	and datetime < "2014-02-24 00:00:00") t  
where 
	stock_prices.code = t.code 
	and stock_prices.datetime >= "2013-05-28 00:00:00" 
	and stock_prices.datetime < "2014-02-24 00:00:00"
	and FLOOR(UNIX_TIMESTAMP(stock_prices.datetime)/86400)
		= FLOOR(1+UNIX_TIMESTAMP(t.datetime)/86400) 
GROUP BY FLOOR(UNIX_TIMESTAMP(stock_prices.datetime)/86400)
```

### IF统计
```
SELECT 
    ccc_news.* , 
    SUM(if(ccc_news_comments.id = 'approved', 1, 0)) AS comments
FROM ccc_news
    LEFT JOIN ccc_news_comments ON ccc_news_comments.news_id = ccc_news.news_id
WHERE `ccc_news`.`category` = 'news_layer2'
    AND `ccc_news`.`status` = 'Active'
GROUP BY ccc_news.news_id
ORDER BY ccc_news.set_order ASC
LIMIT 20 
```

### 数据验证
```
SELECT 
	sdyk_raw.domains.domain, 
	sdyk_raw.domains.domain_name,
	t1.count as project_count,
	t2.count as project_snapshot_count,
	date('2018-09-03 00:00:00') as sd,
	date('2018-09-04 00:00:00') as ed
FROM 
(
	SELECT 
		sdyk_raw.projects.domain_id, count(sdyk_raw.projects.id) as count
	FROM sdyk_raw.projects 
	WHERE sdyk_raw.projects.insert_time >= '2018-09-03 00:00:00'
		AND sdyk_raw.projects.insert_time < '2018-09-04 00:00:00'
	GROUP BY sdyk_raw.projects.domain_id
) AS t1,
(
	SELECT 
		sdyk_raw_snapshot.projects.domain_id, count(sdyk_raw_snapshot.projects.id) as count
	FROM sdyk_raw_snapshot.projects 
	WHERE sdyk_raw_snapshot.projects.insert_time > '2018-09-03 00:00:00'
		AND sdyk_raw_snapshot.projects.insert_time < '2018-09-04 00:00:00' 
	GROUP BY sdyk_raw_snapshot.projects.domain_id
) AS t2,
sdyk_raw.domains
WHERE t1.domain_id = sdyk_raw.domains.id 
	AND t2.domain_id = sdyk_raw.domains.id 
```