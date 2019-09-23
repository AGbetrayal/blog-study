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

