## 基础

### Class.isAssignableFrom与 instanceof 区别
- class1.isAssignableFrom(class2) 判断class2是不是class1的子类或者子接口；
- instanceof 是用来判断一个对象实例是否是一个类或接口的或其子类子接口的实例。

### 子类重写（覆盖）父类的方法必须满足的条件
- 父类中的方法在子类中必须可见，即子类继承了父类中的该方法（可以显式的使用super关键字来访问父类中的被重写的方法），如果父类中的方法为private类型的，那么子类则无法继承，也无法覆盖。
- 子类和父类的方法必须是实例方法，如果父类是static方法而子类是实例方法，或者相反都会报错。 如果父类和子类都是static方法，那么子类隐藏父类的方法，而不是重写父类方法。
- 子类和父类的方法必须要具有相同的函数名称、参数列表，并且子类的返回值与父类相同或者是父类返回类型的子类型（jdk1.5之后）。 如果方法名称相同而参数列表不同（返回类型可以相同也可以不同），那么只是方法的重载，而非重写。 如果方法名称和参数列表相同，返回值类型不同，子类返回值类型也不是父类返回值类型的子类，编译器就会报错。
- 子类方法的访问权限不能小于父类方法的访问权限（可以具有相同的访问权限或者子类的访问权限大于父类）。 访问权限由高到低：public、protected、包访问权限、private。如果子类方法的访问权限低于父类，则编译器会给出错误信息
- 子类方法不能比父类方法抛出更多的编译时异常（不是运行时异常），即子类方法抛出的编译时异常或者和父类相同或者是父类异常的子类。

### 内部类 外部类

* '''内部类定义原则'''：
   - 当描述事物的时候，事物的内部还有事物，该事物用内部类来描述。因为内部类事物在使用外部事物的内容。
* '''内部类访问规则'''
   - 内部类可以直接访问外部类，包括私有(private)。之所有可以直接访问外部类中的成员，是因为内部类中持有了一个外部类的引用，写法为：'''外部类名.this.外部成员名'''。
   - 外部类访问内部类，必须建立内部类对象。

`Outer.Inner in = new Outer().new Inner();`
* 当内部类在成员位置上，就可以被成员修饰符所修饰，比如private，将内部类在外部类中封装。
* static：内部类就具备了静态的特性。称为静态内部类。当内部类被静态修饰后，只能访问外部类中的静态成员，出现了访问局限。
* 直接访问static内部类的非静态成员
   `外部类名.内部类名 oi = new 外部类名.内部类名().function();` // 外部类访问内部类的函数。
* 直接访问static内部类静态成员
   `外部类名.内部类名.function();`
* 当内部类中定义了静态成员，该内部类必须是static。当外部类中的静态方法访问内部类时，内部也必须是static.
* 在方法中的类，为局部内部类，不可以被static修饰。
* 匿名内部类也就是没有名字的内部类，是对内部类的简化。内部类必须继承一个类或接口。

内部类访问外部类对象 
- `OuterClass.this`

## 多线程
### 线程卡死 / 死锁
线程运行需要的'''内存'''、'''锁对象'''等资源不能获取
运行java程序的监控软件，'''jvisualvm'''监控线程运行状态，执行线程'''dump'''操作，查看线程wait的对象

## 网络相关

### URLConnection
在可读取的 InputStream 上调用 close() 函数的时候会污染连接池 (connection pool)。可以通过禁用连接池的方法还解决这个问题：
```java
System.setProperty("http.keepAlive", "false");
```
增加对于特定host的并发量：
```java
System.setProperty("http.maxConnections", 50);
```
HTTP ContentEncoding `deflate`
```java
in = new InflaterInputStream(conn.getInputStream()), new Inflater(true));
```
总之，还是使用Apache HttpClient吧

### Java Https 无法进行Proxy验证
```java
System.setProperty("jdk.http.auth.tunneling.disabledSchemes", "");
System.setProperty("jdk.http.auth.proxying.disabledSchemes", "");
```
或增加JVM参数：
```
-Djdk.http.auth.tunneling.disabledSchemes="" -Djdk.http.auth.proxying.disabledSchemes=""
```
参考：
* https://blog.capsiel.fr/articles/bonnes-pratiques/java/java-proxy/
* https://bugs.openjdk.java.net/browse/JDK-8168839

### block问题
```
"btpool0-41529" prio=10 tid=0x00002aaac45dd000 nid=0x7db9 in Object.wait() [0x0000000047155000]
java.lang.Thread.State: WAITING (on object monitor)
at java.lang.Object.wait(Native Method)
at java.lang.Object.wait(Object.java:485)
at java.net.InetAddress.checkLookupTable(InetAddress.java:1267)
- locked <0x00000006e385b000> (a java.util.HashMap)
at java.net.InetAddress.getAddressFromNameService(InetAddress.java:1190)
```
解决方法：
`-Djava.net.preferIPv4Stack=true`

### java.net.ProtocolException: Server redirected too many times
因为没有维护用户会话，请求在无限循环中重定向。会话通常由一个 cookie 支持。在使用 URLConnection 之前，需要创建一个 CookieManager。
```java
// First set the default cookie manager.
CookieHandler.setDefault(new CookieManager(null, CookiePolicy.ACCEPT_ALL));
// All the following subsequent URLConnections will use the same cookie manager.
URLConnection connection = new URL(url).openConnection();
// ...
connection = new URL(url).openConnection();
```

### PKIX：unable to find valid certification path to requested target
缺少安全证书时出现的异常，将网站安全证书导入运行时的cert keystore

自定义 cert keystore
```java
System.setProperty("javax.net.ssl.trustStore", path_to_your_cacerts_file);
```

### jsch

#### com.jcraft.jsch.JSchException: UnknownHostKey
```java
Properties config = new java.util.Properties(); 
config.put("StrictHostKeyChecking", "no");
session.setConfig(config);
```

## JDBC连接池
### 基本步骤
```java
// 创建连接
Connection conn = PooledDataSource.getDataSource(dbName).getConnection();
// 创建PreparedStatement
PreparedStatement ps = conn.prepareStatement(sql) //防止sql注入
// 结果集ResultSet
ResultSet rs = ps.executeQuery();
// 遍历结果集
```

### JDBC游标处理数据
通过游标将结果集分批进行处理，是结果集的一种扩展。[JDBC利用游标分页查](http://www.blogjava.net/supercrsky/articles/221755.html)
游标通过执行以下操作来扩展结果集处理：
* 允许定位在结果集的特定行。
* 从结果集的当前位置检索一行或一部分行。
* 支持对结果集中当前位置的行进行数据修改。
* 为由其他用户对显示在结果集中的数据库数据所做的更改提供不同级别的可见性支持。

#### ResultSet设置
游标类型
- ResultSet.TYPE_SCROLL_SENSITIVE //只进游标
- ResultSet.TYPE_FORWARD_ONLY //可滚动。但是不受其他用户对数据库更改的影响。
- ResultSet.TYPE_SCROLL_INSENSITIVE //可滚动。当其他用户更改数据库时这个记录也会改变。
游标
- ResultSet.CONCUR_READ_ONLY //只读
- ResultSet.CONCUR_UPDATABL //可更新

```java
public void getData(String sql) throws Exception {

	Connection conn = PooledDataSource.getDataSource(DBName).getConnection();
	PreparedStatement ps = conn.prepareStatement(sql, ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_READ_ONLY);
	// ps.setMaxRows(start+maxCount-1);
 
	ResultSet rs = ps.executeQuery();
	rs.first();
	while(rs.next()) {
		System.err.println(rs.getString(index));
	}
}
```


### 数据库连接获取不到
原因：java程序出现异常时，数据库连接没有成功回收到连接池

解决方法：
- 数据库连接是一种很稀缺的资源，成功申请到数据库连接时，一定要确保连接被回收到数据库连接池中。建议返回数据库连接的代码放到finall{}语句块中。
- 查看log，数据库连接是对称的，一个连接申请到，必然会有一个连接返回到连接池中，打印连接池的状态，查看连接的最大数量和繁忙数量。

### BasicResourcePool.awaitAvailable
长期运行线程无法获取连接问题

可能原因：Connection leak
- 设置: unreturnedConnectionTimeout 超过设定值，没有归还的connections会被销毁并返回连接池
- 设置: debugUnreturnedConnectionStackTraces，捕捉在自动归还超时连接前，使用该连接的堆栈信息
- 参考：http://www.mchange.com/projects/c3p0/#configuring_to_debug_and_workaround_broken_clients

## Turning

### GC
Oracle推荐使用G1 grabage collector（Garbage-First），具体请参考：[Getting Started with the G1 Garbage Collector](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html)

常见配置：
```
-Xms24G -Xmx24G -XX:PermSize=512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=20 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70
-Xms4G -Xmx4G -XX:PermSize=512m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=20 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70
```

参数说明：
* -Xms, -Xmx：堆初始内存和最大内存
* -XX:+UseG1GC：使用G1垃圾回收机制
* -XX:MaxGCPauseMillis：最大GC暂停时间
* -XX:ParallelGCThreads：设定并行阶段的垃圾回收器线程数
* -XX:ConcGCThreads：设定并发垃圾回收器线程数
* -XX:InitiatingHeapOccupancyPercent：设定垃圾回收器启动的堆使用量阈值。这个配置系统默认值为45，如果配置为0则认为是do constant GC cycles

### 导出堆栈信息
```java
jps
jstack {PID} > outfile
jmap -heap {PID} > outfile
```

## 其他
### 绘图
后端直接渲染图表，保护数据
- [XChart](https://github.com/knowm/XChart) | [example code](https://knowm.org/open-source/xchart/xchart-example-code/)