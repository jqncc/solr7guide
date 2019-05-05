## 在Solr中使用JMX 
<div class="content-intro view-box ">Java 管理扩展（Java Management Extensions，JMX）是一种技术，它使复杂的系统能够被工具所控制，而系统和工具之间没有任何相互的了解。从本质上讲，它是一个标准的接口，通过它可以查看和操纵复杂的系统。  
  
Solr和Java组成中的其他优秀成员一样，可以通过JMX接口进行控制。一旦启用，您可以使用JMX客户端（如jconsole）与Solr进行连接。  
如果您不熟悉JMX，您可能会发现以下概述非常有用：http : //docs.oracle.com/javase/8/docs/technotes/guides/management/agent.html。  

## 配置JMX

JMX支持通过定义度量reporter进行配置，如“ JMX Reporter”一节中所述。  
  
如果您在Solr的JVM中运行现有的MBean服务器，或者如果使用系统属性-Dcom.sun.management.jmxremote启动Solr，即使您没有在solr.xml中明确定义reporter，Solr也会在启动时自动识别它的位置。您还可以使用reporter定义中定义的参数定义MBean服务器的位置。  

## 配置MBean服务器

7.0版本之前的Solr定义了在solrconfig.xml中支持JMX。这已被更改为上面定义的标准reporter配置。reporter配置的参数允许定义现有MBean服务器的位置或地址。  
  
通过传递系统参数-Dcom.sun.management.jmxremote，可以在Solr启动时启动MBean服务器。有关可用于启动和控制MBean服务器的其他设置，请参阅Oracle文档：http://docs.oracle.com/javase/8/docs/technotes/guides/management/agent.html。  

### 配置到Solr JMX的远程连接

如果您需要将启用JMX的Java分析工具（如JConsole或VisualVM）附加到远程Solr服务器，则需要在启动Solr服务器时启用远程JMX访问。只需将 solr.in.sh 或 solr.in.cmd（Windows系统中）文件中的 ENABLE_REMOTE_JMX_OPTS 属性更改为 true。您还需要为JMX RMI连接器选择要绑定的端口，例如18983。例如，如果您的Solr包含如下的脚本集：  
  
```
ENABLE_REMOTE_JMX_OPTS=true
RMI_PORT=18983
```
JMX RMI连接器将允许Java分析工具连接到端口18983。启用时，以下属性在启动Solr时传递给JVM：  
```
-Dcom.sun.management.jmxremote \
-Dcom.sun.management.jmxremote.local.only=false \
-Dcom.sun.management.jmxremote.ssl=false \
-Dcom.sun.management.jmxremote.authenticate=false \
-Dcom.sun.management.jmxremote.port=18983 \
-Dcom.sun.management.jmxremote.rmi.port=18983
```
我们不建议在生产环境中启用远程JMX访问，但在进入生产前执行性能和用户验收测试时， 它有时可能会有所帮助。  
  
有关这些设置的更多信息，请参阅：http : //docs.oracle.com/javase/8/docs/technotes/guides/management/agent.html。  
```
注意：将JMX连接到运行在NAT之后的机器（例如，Amazon的EC2服务）并不是一件简单的事情。该java.rmi.server.hostname系统属性可能会有帮助，但是在服务器上运行jconsole 并使用远程桌面通常是最简单的解决方案。请参阅：http://web.archive.org/web/20130525022506/http://jmsbrdy.com/monitoring-java-applications-running-on-ec2-i
```
