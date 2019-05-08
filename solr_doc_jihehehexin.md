# Solr(Collections)集合和核心(core)操作 

在 Solr 中 bin/solr 脚本还可以帮助您创建新的集合（在 SolrCloud 模式下）或核心（在独立模式下）或删除集合。

Solr核心(Core)是Lucene索引的运行实例，包含使用它所需的所有Solr配置文件。Solr应用程序可以包含一个或多个核心


## 创建一个核心或集合

使用 create 命令检测 Solr 正在运行的模式（独立或 SolrCloud），然后根据模式创建一个核心或集合。

```
bin/solr create [options]
bin/solr create -help
```

## 创建核心或集合的参数

-c &lt;name&gt;  
要创建的核心或集合的名称（必需）。示例：bin/solr create -c mycollection

-d &lt;confdir&gt;  
配置目录。这个默认为_default。在 SolrCloud 模式下运行时，请参阅下面的 配置目录和 SolrCloud 部分以获取关于此选项的更多详细信息。示例： bin/solr create -d _default

-n &lt;configName&gt;  
配置名称。这与内核或集合的默认名称相同。示例：bin/solr create -n basic

-p &lt;port&gt;  
发送 create 命令的本地 Solr 实例的端口；默认情况下，脚本尝试通过查找正在运行的 Solr 实例来检测端口。
如果您在同一台主机上运行多个独立的 Solr 实例，则此选项非常有用，因此要求您具体说明要在哪个实例中创建核心。示例：  
bin/solr create -p 8983

-s &lt;shards&gt; 或者 -shards  
分割集合的分片数量，默认值为1；仅适用于以 SolrCloud 模式运行 Solr 的时候。例如：bin/solr create -s 2

-rf &lt;replicas&gt; 或者 -replicationFactor  
集合中每个文档的副本数量。默认值是1（不复制）。例如：bin/solr create -rf 2

-force  
如果尝试以 “root” 用户身份运行 create，则脚本将退出，并显示警告，将 Solr 或针对 Solr 的操作作为 “root” 运行会导致问题。可以用 -force 参数覆盖此警告。例如：bin/solr create -c foo -force


## 配置目录和SolrCloud

在 SolrCloud 中创建集合之前，集合使用的配置目录必须上传到 ZooKeeper。create 命令支持多个用例来说明集合和配置目录的工作方式。您需要做的主要决定是ZooKeeper 中的配置目录是否应在多个集合中共享。  
让我们通过几个例子来说明配置目录在 SolrCloud 中的工作方式。  
首先，如果你没有提供 -d 或者 -n 选项，那么默认的配置（$SOLR_HOME/server/solr/configsets/_default/conf）会被上传到 ZooKeeper，并且使用与集合相同的名称。  
例如，下面的命令将导致 _default 配置被上传到 ZooKeeper 中的 /configs/contacts：bin/solr create -c contacts。  
如果您使用 bin/solr -c contacts2 创建另一个集合，那么 _default 目录的另一个副本将被上传到 ZooKeeper 下的 /configs/contacts2。  
对 "联系人" 集合的配置所做的任何更改都不会影响 contacts2 集合。简而言之，默认行为为您创建的每个集合创建一个配置目录的唯一副本。  

您可以使用该 -n 选项覆盖在 ZooKeeper 中给配置目录指定的名称。例如，命令 bin/solr create -c logs -d _default -n basic 会将server/solr/configsets/_default/conf 目录上传到 ZooKeeper /configs/basic。  
请注意，我们使用该 -d 选项来指定的配置与默认值不同。Solr 在 server/solr/configsets 中提供了几个内置的配置。但是，您也可以使用该 -d 选项提供到您自己的配置目录的路径。比如命令 bin/solr create -c mycoll -d /tmp/myconfigs，将上传 /tmp/myconfigs 到 ZooKeeper 下的 /configs/mycoll。  

重申一下，除非您使用该-n选项覆盖配置目录，否则配置目录将以集合名称命名。  
其他集合可以通过使用 -n 选项指定共享配置的名称来共享相同的配置。例如，以下的命令将创建一个新的集合，该集合共享之前创建的基本配置：bin/solr create -c logs2 -n basic。  

## 数据驱动模式和共享配置

_default 模式可以随着数据的索引而发生变化，因为它具有无模式的功能（即，数据驱动改的模式更改）。因此，我们建议您不要在集合之间共享数据驱动的配置，除非您确定所有集合都应该继承将数据索引到其中一个集合时所做的更改。您可以通过以下方式（假设集合名称为 mycollection）关闭集合的无模式功能（即数据驱动的模式更改）：

```
curl http://host:8983/solr/mycollection/config -d '{"set-user-property": {"update.autoCreateFields":"false"}}'
```


## 删除核心或集合

delete 命令检测 Solr 正在运行的模式（独立或 SolrCloud），然后根据需要删除指定的核心（独立）或集合（SolrCloud）。

```
bin/solr delete [options]
bin/solr delete -help
```

如果以 SolrCloud 模式运行，则 delete 命令将检查您正在删除的集合使用的配置目录是否正在被其他集合使用。如果没有，那么配置目录也会从 ZooKeeper 中删除。  
例如，如果您创建了一个具有 bin/solr create -c contacts 的集合，那么 delete 命令 bin/solr delete -c contacts 将检查 /configs/contacts 配置目录是否被任何其他集合使用。如果不是，则 /configs/contacts 目录从 ZooKeeper 中删除。

## 删除核心或集合参数

-c &lt;name&gt;  
要删除的核心/集合的名称（必填）。例如：bin/solr delete -c mycoll

-deleteConfig  
是否也应该从 ZooKeeper 中删除配置目录。默认是true。如果其他集合正在使用配置目录，则即使您将 deleteConfig 作为 true，也不会删除它。例如：bin/solr delete -deleteConfig false

-p &lt;port&gt;  
要向其发送 delete 命令的本地 Solr 实例的端口。默认情况下，脚本尝试通过查找正在运行的 Solr 实例来检测端口。  
如果您在同一主机上运行多个独立的 Solr 实例，则此选项非常有用，因此要求您具体指定从哪个实例删除该核心。例如： bin/solr delete -p 8983

