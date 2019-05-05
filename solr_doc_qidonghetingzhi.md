# 启动和停止Solr

## 启动和重启

您可以使用 start 命令来启动 Solr，使用 restart 命令允许您在 Solr 已经运行或者已经停止的情况下重新启动 Solr。
start 和 restart 命令有多个参数选项，可以运行在SolrCloud模式，示例，使用非默认的主机名或端口开始并指向本地的 ZooKeeper 集合。

```
bin/solr start [options]
bin/solr start -help
bin/solr restart [options]
bin/solr restart -help
```

使用 restart 命令时，必须传递您在启动 Solr 时最初传递的所有参数。在幕后，启动了一个停止请求，所以 Solr 将在被再次启动之前停止。如果没有节点已经运行，则重新启动将跳过此步骤停止并继续启动 Solr。  

### 启动参数

bin/solr 脚本提供了许多选项，允许您以常见的方式自定义服务器，示例更改侦听端口。但是，大多数默认设置对于大多数 Solr 安装都是足够的，特别是刚开始时。

**-a "&lt;string&gt;"**  
    使用额外的 JVM 参数（示例以 -X 开头的参数）启动 Solr。如果您正在传递以 “-D” 开头的 JVM 参数，则可以省略 -a 选项。示例：

```
bin/solr start -a "-Xdebug -Xrunjdwp:transport=dt_socket, server=y,suspend=n,address=1044"
```

**-cloud**  
     以 SolrCloud 模式启动 Solr，该模式也将启动 Solr 附带的嵌入式 ZooKeeper 实例。这个选项等价于-c。 如果使用外部ZooKeeper，而不是嵌入式（单节点）ZooKeeper，则还应该传递 -z 参数。有关更多详细信息，请参阅下面的 SolrCloud 模式部分。示例：bin/solr start -c

**-d &lt;dir&gt;**  
    定义一个服务器目录，默认为server（如，$SOLR_HOME/server）。重写此选项的情况并不常见。在同一台主机上运行多个 Solr 实例时，更常见的是为每个实例使用相同的服务器目录，并使用 -s 选项使用唯一的Solr主目录更为常见。示例：bin/solr start -d newServerDir

**-e &lt;name&gt;**  
    以一个示例配置启动 Solr。提供这些示例可以帮助您更快速地使用 Solr，或者只是尝试一个特定的功能。 可用的选项是： 
- cloud
- techproducts
- dih
- schemaless

有关示例配置的更多详细信息，请参见下面的使用示例配置运行部分。示例：bin/solr start -e schemaless

**-f**  
在前台启动 Solr；在使用 -e 选项运行示例时，不能使用此选项。示例：bin/solr start -f

**-h &lt;hostname&gt;**  
用定义的主机名启动 Solr。如果没有指定，将假定为 'localhost'。示例：bin/solr start -h search.mysolr.com

**-m &lt;memory&gt;**  
以定义的值启动 Solr ：JVM 的 min（-Xms）和 max（-Xmx）堆大小。示例：bin/solr start -m 1g

**-noprompt**  
启动 Solr 并禁止任何可能出现的提示。这会隐含地接受所有默认的副作用。  
示例，使用“cloud”示例时，交互式会话将指导您完成 SolrCloud 集群的多个选项。如果您想接受所有的默认设置，您可以简单地将 -noprompt 选项添加到您的请求中。示例：bin/solr start -e cloud -noprompt

**-p &lt;port&gt;**  
在定义的端口上启动 Solr。如果没有指定，将使用 “8983”。示例：bin/solr start -p 8655

**-s &lt;dir&gt;**  
设置 solr.solr.home 系统属性；Solr 将在这个目录下创建核心目录。这允许您在同一主机上运行多个 Solr 实例，同时使用 -d 参数重新使用相同的服务器目录集。如果设置，则除非 ZooKeeper 中存在 solr.xml，否则指定的目录应包含 solr.xml 文件。默认值是server/solr。  
  运行示例（-e）时忽略此参数，因为 solr.solr.home 取决于运行哪个示例。示例：bin/solr start -s newHome

**-v**  
 将log4j的日志记录级别从INFO更改为DEBUG，具有与log4j.properties相应编辑时相同的效果. 示例：bin/solr start -f -v

**-q**  
将log4j的日志记录级别从INFO更改为WARN。具有与log4j.properties相应编辑时相同的效果。这在您希望将日志记录限制为警告和错误的生产设置中非常有用.示例：bin/solr start -f -q

**-V**  
用启动脚本中的详细消息启动 Solr。 示例：bin/solr start -V

**-z &lt;zkHost&gt;**  
指定外部zookeeper地址。此选项仅用于 -c 选项，以SolrCloud模式启动Solr。如果未提供此选项，Solr 将启动嵌入式ZooKeeper实例，并将该实例用于 SolrCloud 操作。 示例：

```
bin/solr start -c -z server1:2181,server2:2181
```


**-force**  
强制使用root身份启动,默认如果尝试以root用户身份启动Solr，脚本将退出，并显示警告，将Solr作为“root”运行可能会导致问题。可以用 -force 参数覆盖此警告。 示例：sudo bin/solr start -force


为强调默认设置的工作原理，请花点时间了解以下命令是等效的：

```
bin/solr start
bin/solr start -h localhost -p 8983 -d server -s solr -m 512m
```

如果默认设置适合您的需要，则无需在启动时定义所有选项。

### 设置Java系统属性

bin/solr 脚本将向 JVM 传递以 -D 开头的任何附加参数，从而允许您设置任意的 Java 系统属性。
      
示例, 将自动 soft-commit 频率设置为3秒，可以执行以下操作:  
```
bin/solr start -Dsolr.autoSoftCommit.maxTime=3000
```


### SolrCloud 模式

-c 和 -cloud 选项是相同的：

```sh
bin/solr start -c
bin/solr start -cloud
```

如果你指定一个 ZooKeeper连接地址，示例：-z 192.168.1.4:2181，那么 Solr将连接到 ZooKeeper并加入集群。  
如果在 cloud 模式下启动 Solr 时没有指定 -z 选项，Solr 将启动一个内嵌的ZooKeeper服务器监听 Solr 端口 + 1000，也就是说，如果 Solr 在端口 8983 上运行，则嵌入式 ZooKeeper将监听端口 9983。
>注意：如果您的 ZooKeeper 连接字符串使用chroot(如 localhost:2181/solr)，则需要在使用 bin/solr 脚本启动SolrCloud之前创建/solr znode,+要执行此操作，请使用mkroot下面概述的命令，例如：bin/solr zk mkroot /solr -z 192.168.1.4:2181

在 SolrCloud 模式下启动时，交互式脚本会话将提示您选择一个要使用的 configset。 有关在 SolrCloud 模式下启动 Solr 的更多信息，另请参阅SolrCloud入门部分。

### 使用示例配置运行

```sh
bin/solr start -e <name>;
```

示例配置允许您快速启动配置，它反映了您希望用 Solr 完成的配置。  
每个示例都使用托管模式启动 Solr，该模式允许使用 Schema API 进行模式编辑，但不允许手动编辑模式文件。  
如果您希望直接手动修改 schema.xml 文件，则可以按照 SolrConfig 中的 “Schema Factory”定义中描述的那样更改这个默认值。  
除非在下面的描述中另有说明，否则这些示例不会启用 SolrCloud 或 schemaless 模式。  
提供了以下示例：  

- **cloud**：这个例子在一台机器上启动一个 1-4 节点的 SolrCloud 集群。选择后，交互式会话将开始指导您选择要使用的初始配置集的选项，示例群集的节点数量、要使用的端口以及要创建的集合的名称。当使用这个例子时，你可以从 $SOLR_HOME/server/solr/configsets 中找到的任何可用的配置集中进行选择。
- **techproducts**：本示例以独立模式启动 Solr，并为 $SOLR_HOME/example/exampledocs 目录中包含的示例文档设计模式。所使用的 configset 可以在$SOLR_HOME/server/solr/configsets/sample_techproducts_configs 中找到。
- **dih**：本示例在独立模式下启动 Solr，并启用了 DataImportHandler（DIH），并为 DIH 支持的不同类型的数据（如数据库内容、电子邮件、RSS 提要等）预先配置了几个 dataconfig.xml 示例文件。所使用的 configset 是定制的 DIH，并且在 $SOLR_HOME/example/example-DIH/solr/conf 被发现。有关 DIH 无: 本示例使用托管架构在独立模式下启动 Solr, 如 SolrConfig 中的 "架构工厂定义" 中所述, 并提供极少的预定义架构。solr 将以此配置运行在无模式下, solr 将在模式中创建字段, 并猜测在传入文档中使用的字段类型。所使用的 configset 可在 $SOLR _home/服务器/SOLR/configsets/_default 中找到。的更多信息，请参阅使用数据导入处理程序上载结构化数据存储数据一节。
- **schemaless**：本示例使用托管模式以独立模式启动 Solr，如 SolrConfig 中的模式工厂定义部分所述，并提供了一个非常简单的预定义模式。Solr 将以这种配置在schemaless 模式下运行，其中 Solr 将在模式中动态创建字段，并将猜测传入文档中使用的字段类型。所使用的 configset 可以在 $SOLR_HOME/server/solr/configsets/_default 中找到。

>注意：运行 in-foreground 选项（-f）与 -e 选项不兼容，因为在启动 Solr 服务器后，脚本需要执行其他任务。


## 停止 Solr

stop 命令向正在运行的 Solr 节点发送 STOP 请求，从而使其正常关闭。**该命令将等待180秒，以便 Solr 正常停止，然后强制终止进程（kill -9）**。

```sh
bin/solr stop [options]
bin/solr stop -help
```

### 停止参数

**-p &lt;port&gt;**   
 停止在给定端口上运行Solr。如果您正在运行多个实例，或者正在以 SolrCloud 模式运行，则需要在单独的请求中指定端口或使用 -all 选项。  示例：bin/solr stop -p 8983

**-all**  
停止所有正在运行的具有有效 PID 的 Solr 实例。示例：bin/solr stop -all

**-k &lt;key&gt;**  
停止键用于防止无意中停止 Solr；默认是 “solrrocks”。示例： bin/solr stop -k solrrocks


