# JDBC With Journal
### 由上编可以知道,activemq的存储已从kahaDB转到mysql中了, 但每次发送消息都从mysql获取, 消费后再删除,这过程很繁琐,并且JDBC Store性能不高.如果在一定范围内的新增和消费消息影响不大,但并发量上来了那明显是不行的
```
activemq官方为我们提供了JDBC With Journal.为了在ActiveMQ V4.x中实现持久消息传递的高性能，我们强烈建议您使用我们的高性能日志 - 默认情况下已启用。
这很像一个数据库消息（以及transcation提交/回滚和消息确认）以尽可能快的速度写入日志 - 然后每隔一段时间我们将日志检查到长期持久性存储（在本例中为JDBC）。
它在使用队列时很常见，例如消息在发布后很快消耗掉; 因此，您可以发布10,000条消息，并且只有一些未完成的消息 - 因此，当我们检查JDBC数据库时，我们通常只有
少量消息可以实际写入JDBC。即使我们必须将所有消息写入JDBC，我们仍然可以通过日志获得性能提升，因为我们可以使用大型事务批处理将消息插入JDBC数据库以提高
JDBC端的性能。
JDBC With Journal方式克服了JDBC Store的不足，使用快速的缓存写入技术，大大提高了性能
```

### 使用JDBC With Journal替换jdbc也很简单
```
<persistenceAdapter>
            <!-- <kahaDB directory="${activemq.data}/kahadb"/> --> 
           <jdbcPersistenceAdapter dataSource="#mysql-ds" createTablesOnStartup="true" />
</persistenceAdapter>
```
### 改成
```
<persistenceFactory>
  <journalPersistenceAdapterFactory
      journalLogFiles="4"
      journalLogFileSize="32768"
      useJournal="true"
      useQuickJournal="true"
      dataSource="#mysql-ds"
      dataDirectory="activemq-data"/>
</persistenceFactory>
```
### activemq 停止(./activemq stop)/重启( ./activemq start)
### 启动正常就成功了

## 注意记得在操作系统中别带"_", 会报错

## JDBC Store和JDBC Message Store with ActiveMQ Journal的区别
```
1.JDBC with journal的性能优于jdbc
2.JDBC用于master/slave模式的数据库分享
3.JDBC with journal不能用于master/slave模式
4.一般情况下，推荐使用jdbc with journal
```

