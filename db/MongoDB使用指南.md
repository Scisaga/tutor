## 安装
参考：https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

### 使用Docker
```shell
docker pull mongo
mkdir -p /data/mongodb
docker run -p 27017:27017 --name mongodb --restart always -v /data/mongodb/:/data/db -d mongo:latest
```
参考：
* https://hub.docker.com/_/mongo/

### 启动
```shell
mongod --fork --config /etc/mongod.conf
```

## 用户管理
### 添加管理账户
```shell
use admin
db.createUser(
  {
    user: "AdminSammy",
    pwd: "AdminSammy'sSecurePassword",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```
参考：https://docs.mongodb.com/manual/reference/method/db.createUser/

有可能需要下述方法实现验证
```shell
db.auth( <username>, <password> )
```
参考：https://docs.mongodb.com/manual/reference/method/db.auth/

## 配置与启动
### windows
添加 --auth 参数
```shell
C:\App\MongoDB\Server\3.2\bin\mongod.exe --auth --dbpath C:\App\MongoDB\Server\3.2\db
```
参考：https://docs.mongodb.com/manual/tutorial/enable-authentication/

### ubuntu
```shell
# 下载
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.0.15.tgz
# 解压缩，并创建data和log目录
```

编辑mongod.conf文件
```
# 日志文件位置
logpath=/mongo/replset/log/mongo.log

# 以追加方式写入日志
logappend=true

# 是否以守护进程方式运行
fork = true

#replSet = repset
# 默认27017
port = 27017

#storageEngine=mmapv1
# 数据库文件位置
dbpath=/mongo/replset/data

# 启用定期记录CPU利用率和 I/O 等待
cpu = true

# 是否以安全认证方式运行，默认是不认证的非安全方式
#noauth = true
auth = true

# 静默输出
quiet = true

# 设置每个数据库将被保存在一个单独的目录
directoryperdb = true
```

在命令行启动mongod server
```shell
./bin/mongod --config mongod.conf
```
添加并验证用户：`db.auth("tetra","tetra")`

## 高级应用场景

### 限制内存使用
```shell
apt-get install cgroups
cgcreate -g memory:DBLimitedGroup
echo 16G > /sys/fs/cgroup/memory/DBLimitedGroup/memory.limit_in_bytes
sync; echo 3 > /proc/sys/vm/drop_caches
cgclassify -g memory:DBLimitedGroup '''mongod-pid'''
```

* https://www.percona.com/blog/2015/07/01/using-cgroups-to-limit-mysql-and-mongodb-memory-usage/
* [使用cgroups限制MongoDB的内存使用](http://www.udpwork.com/item/14426.html )

### WARNING: You are running on a NUMA machine.
在原启动命令前面加 `numactl –interleave=all`
```shell
numactl --interleave=all ${MONGODB_HOME}/bin/mongod --config conf/mongodb.conf
```

### WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
需要hugepage在启动时预先分配，而不是动态分配
```shell
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

# 或写入 /etc/rc.local 文件中
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then  
echo never > /sys/kernel/mm/transparent_hugepage/enabled  
fi  
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then  
echo never > /sys/kernel/mm/transparent_hugepage/defrag  
fi
```
参考：
* http://blog.csdn.net/csfreebird/article/details/49307935

## 安全退出
```shell
# 使用root账户登录终端
mongo admin -uroot -p
# 执行退出命令
user admin
db.shutdownServer()
```

## 数据导入导出

### 导出
windows下参数有所变化 /d /u /p /o, 具体使用--help参数查询
```shell
mongodump -d <database_name> -o <directory_backup>
# 特定数据库
mongodump --db mongodevdb --username mongodevdb --password YourSecretPwd
# 特定Collection
mongodump --collection employee --db mongodevdb --username mongodevdb --password YourSecretPwd
# 远程数据库实例
mongodump --host 192.168.1.2 --port 37017 --db mongodevdb --username mongodevdb --password YourSecretPwd
# 归档压缩
mongodump --archive=test.20150715.gz --gzip --db test 
```

### 恢复，首先要终止mongod进程
```shell
mongorestore -d <database_name> <directory_backup>
mongorestore --dbpath /var/lib/mongo --db mongodevdb dump/mongodevdb
# 删除原有数据
mongorestore --dbpath /var/lib/mongo --db mongodevdb --drop dump/mongodevdb
mongorestore --host 192.168.1.2 --port 3017 --db mongodevdb --username mongodevdb --password YourSecretPwd --drop /backup/dump
# 副本 解压缩归档文件
mongorestore --host "replSet/rep1.example.net:27017,rep2.example.net:27017,rep3.example.net:27017" --username user --password --gzip --db test /home/data/mongobak
```

如果导出特定库失败，命令行上加上 `--authenticationDatabase admin`

参考
* http://www.thegeekstuff.com/2013/09/mongodump-mongorestore
* https://stackoverflow.com/questions/31993168/cant-create-backup-mongodump-with-db-authentication-failed
* [How To Back Up, Restore, and Migrate a MongoDB Database on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-back-up-restore-and-migrate-a-mongodb-database-on-ubuntu-14-04)

## 常用操作

### 排序和自然排序`.sort()`

```shell
# 其中1为正序，-1为倒序
db.foo.find().sort({x:1});

# 使用自然排序获取最新插入的记录
db.foo.find().sort({$natural:-1});
db.foo.find().sort({$natural:-1}).limit(50);
```

## 客户端

### Web管理界面 ===
mongoclient
*可进行Docker部署的MongoDB Web管理界面，界面设计较为突出
* https://github.com/nosqlclient/nosqlclient
* https://www.mongoclient.com/

genghis
* 基于Ruby/PHP的MongoDB Web管理界面
* http://genghisapp.com/
* https://github.com/bobthecow/genghis/

mongo-express
* 基于Node.js的MongoDB Web管理界面
* https://github.com/mongo-express/mongo-express

### Java客户端
MongoDB Java Driver
* http://mongodb.github.io/mongo-java-driver/
* https://github.com/mongodb/mongo-java-driver/

Jongo
* 提供了基于shell查询方式的Java接口，以及对象映射方法
* https://github.com/bguerout/jongo
* `friends.find("{age: {$gt: 18}}").as(Friend.class)`

Morphia
* Transparently map your Java entities to MongoDB documents and back.
* http://mongodb.github.io/morphia/

RestHeart
* 基于Java的MongoDB Restful API server
* 可以进行深度整合
* http://restheart.org/

### 云端管理 / 服务器集群管理
MongoDB Atlas
* https://www.mongodb.com/cloud/atlas

MongoDB Cloud Manager
* 可视化监控服务器集群运行状态，Query优化
* https://www.mongodb.com/cloud/cloud-manager

### 桌面客户端
Studio 3T
* 目前来讲做的最完善的MongoDB客户端
* https://studio3t.com/

Mongo Management Studio2.0
* http://mms.litixsoft.de/index.php?lang=en

### 分析器
variety
* 基于Javascript的Schema分析器，例如分析docments中各个字段的填充占比
* https://github.com/variety/variety

## 参考
* http://mongodb-tools.com/
* [MongoDB使用小结：一些常用操作分享](http://www.cnblogs.com/cswuyg/p/4595799.html)