## Solr：设置一个外部ZooKeeper集合 
<div class="content-intro view-box ">虽然Solr与Apache ZooKeeper捆绑在一起，但您应该认为自己不建议在生产环境中使用这个内部的ZooKeeper。  
  
关闭一个冗余的Solr实例也会关闭它的ZooKeeper服务器，这可能不是那么多余的。由于ZooKeeper集合必须有超过半数的服务器在任何给定的时间运行，这可能是一个问题。  
解决这个问题的方法是建立一个外部的ZooKeeper集合。幸运的是，虽然这个过程由于强大的选项数量而显得吓人，但建立一个简单的集合实际上是非常简单的，如下所述。  
## 有多少ZooKeepers？
在ZooKeeper指南（http://zookeeper.apache.org/doc/r3.4.10/zookeeperAdmin.html）中提到：为ZooKeeper服务处于活动状态，必须有大多数的 non-failing（非故障）机器，可以相互沟通。要创建一个能够容忍 F 计算机失败的部署，您应该依靠部署2xF + 1机器进行计数。因此，由三台计算机组成的部署可以处理一个故障，部署五台计算机可以处理两个故障。请注意，部署六台机器只能处理两次故障，因为三台机器不占多数。因此，ZooKeeper的部署通常由奇数个的机器组成。  
在规划配置多少个ZooKeeper节点时，请记住，ZooKeeper集成的主要原则是维护大部分服务器来处理请求。这个多数也被称为“法定人数（quorum）”。  
  
通常建议在您的集合中有一个奇怪的ZooKeeper服务器，所以大部分都是维护的。  
例如，如果您只有两个ZooKeeper节点，而其中一个出现故障，则50％的可用服务器不是多数，所以ZooKeeper将不再提供请求。但是，如果有三个ZooKeeper节点，并且有一个出现故障，则有66％的可用服务器可用，而ZooKeeper将正常继续，而您正在修复一个向下节点。如果有5个节点，则必要时可以继续使用两个向下的节点进行操作。  
有关ZooKeeper集群的更多信息，请参阅http://zookeeper.apache.org/doc/r3.4.10/zookeeperAdmin.html#sc_zkMulitServerSetup上的ZooKeeper文档。  
## 下载Apache ZooKeeper
设置Apache ZooKeeper的第一步当然是下载软件，您可以从http://zookeeper.apache.org/releases.html中获得。  
当使用独立的ZooKeeper时，您需要注意使用Solr发布的最新版本来更新ZooKeeper的版本。由于您将其作为独立应用程序使用，因此升级Solr时不会升级。Solr目前使用Apache ZooKeeper v3.4.10。  
## <span style="font-family: inherit; font-size: 16px; font-weight: 600;">创建一个独立的ZooKeeper</span>

### 创建实例
创建实例是将文件解压缩到特定目标目录的简单方法。实际的目录本身并不重要，只要你知道它在哪里，以及你想在哪里存放它的内部数据。  
### 配置实例
下一步是配置你的ZooKeeper实例。为此，请创建以下文件：&lt;ZOOKEEPER_HOME&gt;/conf/zoo.cfg。对此文件添加以下信息：  
```
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```
参数如下：  
**tickTime**
ZooKeeper的一部分功能是确定哪些服务器在任何给定的时间运行，并且最短的会话时间被定义为两个“ticks”。该<code>tickTime</code>参数以毫秒为单位指定每个刻度应该有多长时间。  
**dataDir**
这是ZooKeeper将存储关于集群的数据的目录。这个目录应该从空白处开始。  
**clientPort**
这是Solr将访问ZooKeeper的端口。  

一旦这个文件就位，你就可以开始ZooKeeper实例了。  
### 运行实例
要运行实例，可以简单地使用ZOOKEEPER_HOME/bin/zkServer.sh提供的脚本，如下所示：zkServer.sh start  
  
同样，ZooKeeper通过附加的配置提供了很大的能力，但是深入研究这些超出了本文档的范围，所以不多做介绍。但是，对于本示例，默认值是正常的。  
### 在实例中指向Solr
在您创建的ZooKeeper实例中指向Solr是使用bin / solr脚本时使用-z参数的简单方法。例如，为了将Solr实例指向您已经在端口2181上启动的ZooKeeper，您需要执行以下操作：  
  
使用已经在端口2181上运行（所有其他的默认值）的ZooKeeper来启动cloud例子：  
```
bin/solr start -e cloud -z localhost:2181 -noprompt
```
在端口2181上添加指向现有ZooKeeper的节点：  
```
bin/solr start -cloud -s &lt;path to solr home for new node&gt; -p 8987 -z localhost:2181
```
<h3>关闭ZooKeeper  
</h3>要关闭ZooKeeper，请使用带“stop”命令的zkServer脚本：zkServer.sh stop。  
## 建立一个ZooKeeper集合
使用外部ZooKeeper集成，与“入门（Getting Started）”示例相比，您需要更仔细地设置一些东西。  
  
不同之处在于，不是简单地启动服务器，而是需要先配置它们以便相互了解和交流。所以您的原始zoo.cfg文件可能是这样的：  
```
dataDir=/var/lib/zookeeperdata/1
clientPort=2181
initLimit=5
syncLimit=2
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
```
在这里你看到三个新的参数：  
**initLimit**
允许追随者连接和同步到leader的时间量 (以刻度为间隔)。在这种情况下，您有5刻度，每一个都是2000毫秒长，因此服务器将等待10秒，以便与leader连接并同步。  
**syncLimit**
时间量（以时钟周期为单位），以允许关注者与ZooKeeper同步。如果追随者落后于领导者，他们将被抛弃。  
**server.X**
这些是集合中所有服务器的ID和位置，它们是彼此通信的端口。服务器ID必须另外存储在<code>&lt;dataDir&gt;/myid</code>文件中，并位于每个ZooKeeper实例中的<code>dataDir</code>。ID标识每个服务器，所以在第一个实例的情况下，您将创建内容为“1” 的<code>/var/lib/zookeeperdata/1/myid</code>文件。  

现在，虽然Solr需要创建全新的目录来运行多个实例，您只需要一个新的ZooKeeper实例，即使它们位于同一台计算机上测试的目的，是一个新的配置文件。为了完成这个例子，你将创建两个更多的配置文件。  
  
该&lt;ZOOKEEPER_HOME&gt;/conf/zoo2.cfg文件应该具有以下内容：  
```
tickTime=2000
dataDir=/var/lib/zookeeperdata/2
clientPort=2182
initLimit=5
syncLimit=2
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
```
您还需要创建&lt;ZOOKEEPER_HOME&gt;/conf/zoo3.cfg：  
```
tickTime=2000
dataDir=/var/lib/zookeeperdata/3
clientPort=2183
initLimit=5
syncLimit=2
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
```
最后，在每个dataDir目录中创建您的myid文件，这样每个服务器就知道它是哪个实例。每台机器上的myid文件中的id必须与“server.X”定义相匹配。因此，上例中名为“server.1”的ZooKeeper实例（或计算机）必须具有包含值“1” 的myid文件。该myid文件可以是1到255之间的任何整数，并且必须与zoo.cfg文件中分配的服务器标识相匹配。  
  
要启动服务器，您可以简单地明确引用配置文件：  
```
cd &lt;ZOOKEEPER_HOME&gt;
bin/zkServer.sh start zoo.cfg
bin/zkServer.sh start zoo2.cfg
bin/zkServer.sh start zoo3.cfg
```
一旦这些服务器正在运行，您可以像以前一样从Solr中引用它们：  
```
bin/solr start -e cloud -z localhost:2181,localhost:2182,localhost:2183 -noprompt
```
## 保护ZooKeeper连接
您可能还希望保护ZooKeeper和Solr之间的通信。  
  
要设置znode的ACL保护，请参阅ZooKeeper访问控制。  
