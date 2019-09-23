## 认识kahaDB
### 简单来说kahaDB就是activemq内置的数据存储,在conf/activemq.xml 配置文件可以看到数据的存储目录
```
<persistenceAdapter>
            <kahaDB directory="${activemq.data}/kahadb"/> 
</persistenceAdapter>
```
### 由此可以看到数据存储的是在 apache-activemq-5.15.9/data/kahadb
## 简单介绍kahaDB
```
 ActiveMQ 5.3以后，出现了KahaDB。她是一个基于文件支持事务的消息存储器，是一个可靠，高性能，可扩展的消息存储器。
 她的设计初衷就是使用简单并尽可能的快。KahaDB的索引使用一个transaction log，并且所有的destination只使用一个index，有人测试表明：如果用于生产环境，支持1万个active connection，每个connection有一个独立的queue。该表现已经足矣应付大部分的需求。
 KahaDB内部分为：data logs, 按照Message ID高度优化的索引，memory message cache。
 data logs扮演一个message journal，存储消息和命令。当大小超过了规定，将会新建一个data log
 .所有在data log里的消息是引用计数的，所以当一个log里的消息不在需要了，可以被删除或者放入archived文件夹。每次消息的写入都是在log的末尾增加记录，所以存储速度很快。
缓存则是临时持有那些有对应消费者在线的消息。如果消费者反馈消息已经成功接收，那么这些消息就不用写入磁盘。
BTree索引，保存消息的引用，并按照message ID排序。Redo log是用来保证MQ broker未干净关闭情况下，用于Btree index的重建。
KahaDB的目录会在你启动MQ后自动创建（使用了KAhaDB作为存储器），
 db log files：以db-递增数字.log命名。
 archive directory: 当配置支持archiving(默认不支持)并且存在，该文件夹才会创建。用于存储不再需要的data logs。
 db.data：存储btree索引
 db.redo：用于hard-stop broker后，btree索引的重建
```

## 这就会引起一个问题: 如果我的activemq出了严重的问题,那么那么我能通过kahaDB进行数据恢复吗? 
### 答案是显然的, 由于kahaDB是activemq内置的, 当activemq出了严重的问题, 数据是很难通过kahaDB进行恢复的,有点高耦合的味道, 这些我们能想到activemq的开发者们当然也想到这个问题,所以activemq的数据不单只能存储在内置的kahaDB中也能存储在外部的数据存储具
### 工具中比如mysql..等等, 但由于我使用的就是mysql所以这里只会将kahaDB 配置成mysql, 如有需要自行上官网查找


## activemq的数据存储由kahaDB置换成mysql
## 要连接mysql记得要把相应的数据库连接jar包放到该目录下: apache-activemq-5.15.9/lib, 该包一般为存放项目依赖的jar包,这里我们放入mysql-connector-java-5.1.9.jar, commons-dbcp-1.4.jar, commons-pool-1.6.jar
### 由上可知kahaDB 的配置是在conf/activemq.xml中
```
<persistenceAdapter>
            <kahaDB directory="${activemq.data}/kahadb"/> 
</persistenceAdapter>
```
### 换成
```
 <persistenceAdapter>
           <!-- <kahaDB directory="${activemq.data}/kahadb"/> -->
           <jdbcPersistenceAdapter dataSource="#mysql-ds" createTablesOnStartup="true" />
</persistenceAdapter>
```
### 接着在标签beans 中添加一下代码(以下的是我的配置,自行修改)
```
 <bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
         <property name="driverClassName" value="com.mysql.jdbc.Driver"/> 
        <!-- <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>-->
        <property name="url" value="jdbc:mysql://192.168.61.129:3306/activemq?relaxAutoCommit=true"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

```

### 由于连接的数据库是activemq, 记得要在自己的mysql工具中新建activemq数据库
### 关闭activemq(./activemq stop),然后重启activemq (./activemq strat)
### 如果在activemq数据库中新增了activemq_acks, activemq_lock, activemq_msgs三张表, 恭喜, 替换成功, 如果失败记得去logs目录看日志, 80%的错误都可以在日志中看到

### 验证: 通过代码发送消息看是否在activemq_msgs表中有显示, 发送的消息都会记得到这张表, 当消息消费后会从该表删除记得 
### 初步介绍下这三张表
```
1：activemq_acks：用于存储订阅关系。如果是持久化Topic，订阅者和服务器的订阅关系在这个表保存。
主要的数据库字段如下：
container：消息的destination 
sub_dest：如果是使用static集群，这个字段会有集群其他系统的信息 
client_id：每个订阅者都必须有一个唯一的客户端id用以区分 
sub_name：订阅者名称 
selector：选择器，可以选择只消费满足条件的消息。条件可以用自定义属性实现，可支持多属性and和or操作 
last_acked_id：记录消费过的消息的id。

----
2：activemq_lock：在集群环境中才有用，只有一个Broker可以获得消息，称为Master Broker，其他的只能作为备份等待Master Broker不可用，才可能成为下一个Master Broker。这个表用于记录哪个Broker是当前的Master Broker。

----
3：activemq_msgs：用于存储消息，Queue和Topic都存储在这个表中。
主要的数据库字段如下：
id：自增的数据库主键 
container：消息的destination 
msgid_prod：消息发送者客户端的主键 
msg_seq：是发送消息的顺序，msgid_prod+msg_seq可以组成jms的messageid 
expiration：消息的过期时间，存储的是从1970-01-01到现在的毫秒数 
msg：消息本体的java序列化对象的二进制数据 
priority：优先级，从0-9，数值越大优先级越高 
activemq_acks用于存储订阅关系。如果是持久化topic，订阅者和服务器的订阅关系在这个表保存。

```
















