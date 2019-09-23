# tcp与nio
### nio是基于tcp的连接协议, NIO采取通道（Channel）和缓冲区(Buffer)来传输和保存数据，它是非阻塞式的I/O，即在等待连接、读写数据（这些都是在一线程以客户端的程序中会阻塞线程的操作）的时候，程序也可以做其他事情，以实现线程的异步操作
### 就连activemq官方的新特性中也推荐使用auto + nio
## 那么在activemq中怎么使用nio呢? 因为在activemq中默认只提供了5中协议: openwire(tcp),amqp,stomp,mqtt,ws.
### 进入conf目录找到activemq.xml 文件, 所有的传输协议都在该标签先的 
```
<transportConnectors>,,, </transportConnectors>
```
### ,所有把所需的协议添加到该标签里即可
### 结合官方推荐和公司项目需要,我修改成
```
<transportConnector name="auto+nio" uri="auto+nio://0.0.0.0:61111?trace=true"/> 
```
### 变成
```
<transportConnectors>
            <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
             <transportConnector name="auto+nio" uri="auto+nio://0.0.0.0:61111?trace=true"/> 
        </transportConnectors>
```
### ,添加到该标签下,记得要重启才能生效,记得要重启才能生效,记得要重启才能生效, 实验成功

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
