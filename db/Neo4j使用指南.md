## 基本概念
* Node
* Relation
* Properties
* [Graph database concepts](https://neo4j.com/docs/getting-started/current/graphdb-concepts/)
* https://github.com/neo4j/neo4j
* https://neo4j.com/developer/get-started/

## 安装部署
```shell
cd /opt \
&& wget https://neo4j.com/artifact.php?name=neo4j-community-3.5.17-unix.tar.gz \
&& tar -xvf 'artifact.php?name=neo4j-community-3.5.17-unix.tar.gz' && mv neo4j-community-3.5.17 neo4j
```

修改配置 `vi /opt/neo4j/conf/neo4j.conf`
```shell
# 反注释
dbms.connectors.default_listen_address=0.0.0.0
```

启动
```shell
cd /opt/neo4j && bin/neo4j start
```

参考
* [neo4j 配置](http://weikeqin.com/2017/04/05/neo4j-config/)

## Java
* https://neo4j.com/developer/java/

### neo4j-ogm
使用Annotation方式对POJO Java类进行标注，方便程序化数据持久化/查询
* https://github.com/neo4j/neo4j-ogm
   ```
   dependencies {
     compile 'org.neo4j:neo4j-ogm-core:3.2.11'
     compile 'org.neo4j:neo4j-ogm-http-driver:3.2.11'
     compile 'org.neo4j:neo4j-ogm-bolt-driver:3.2.11'
     compile 'org.neo4j:neo4j-ogm-embedded-driver:3.2.11'
   }
   ```

### 示例项目
* https://github.com/neo4j-examples/neo4j-movies-java-bolt
   - 使用Cypher进行Neo4j查询，使用Spark-Java作为接口服务端，前端展示使用jquery/bootstrap/d3
* https://github.com/neo4j-examples/movies-java-jdbc
