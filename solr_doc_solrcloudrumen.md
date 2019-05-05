## SolrCloud入门 
<div class="content-intro view-box ">SolrCloud旨在提供高度可用的容错环境，用于在多个服务器之间分发索引内容和查询请求。  
  
这是一个将数据组织成可以托管在多台机器上的多个碎片的系统，其中副本为可扩展性和容错性提供了冗余，还有一个ZooKeeper服务器，用于帮助管理整个结构，以便索引和搜索请求可以正确地路由。  
本节详细介绍了SolrCloud及其内部工作原理，但是在深入研究之前，最好先了解一下您要完成的任务。  
这个页面提供了一个在SolrCloud模式下启动Solr的简单教程，所以您可以开始了解在索引和提供查询期间碎片如何相互影响。为此，我们将使用在单台机器上配置SolrCloud的简单示例，这显然不是真正的生产环境，其中包括几台服务器或虚拟机。在实际的生产环境中，您还将使用真正的机器名称而不是我们在此使用的“localhost”。  
在本节中，您将学习如何使用启动脚本和特定的配置集来启动SolrCloud集群。  
本教程假定您已经熟悉了使用 Solr 的基本知识。如果您需要进行复习，请参阅入门部分以了解Solr概念。如果您将文档作为该练习的一部分进行加载，则应该从这些SolrCloud教程的全新Solr安装开始。  

## SolrCloud示例

### 交互式启动

该bin/solr脚本使您可以轻松开始使用SolrCloud，因为它引导您完成在云模式下启动Solr节点并添加集合的过程。要开始，只需：  
```
bin/solr -e cloud
```
这将启动一个交互式会话，引导您完成设置嵌入式ZooKeeper的简单SolrCloud集群的步骤。  
  
该脚本首先询问您要在本地群集中运行多少个Solr节点，默认值为2。  
```
Welcome to the SolrCloud example!
This interactive session will help you launch a SolrCloud cluster on your local workstation.
To begin, how many Solr nodes would you like to run in your local cluster? (specify 1-4 nodes) [2]
```
该脚本支持最多启动4个节点，但我们建议在启动时使用默认值2。这些节点将分别存在于一台机器上，但将使用不同的端口来模拟不同服务器上的操作。  
  
接下来，该脚本会提示您将端口绑定到每个Solr节点，例如：  
```
 Please enter the port for node1 [8983]
```
为每个节点选择任何可用的端口；第一个节点的默认值是第二个节点的8983和7574。该脚本将按顺序启动每个节点，并向您显示它用于启动服务器的命令，例如：  
```
solr start -cloud -s example/cloud/node1/solr -p 8983
```
第一个节点也将启动一个绑定到端口9983的嵌入式ZooKeeper服务器。第一个节点的Solr主目录在example/cloud/node1/solr中，如-s选项所示。  
  
启动集群中的所有节点后，脚本会提示您输入要创建的集合的名称：  
```
 Please provide a name for your new collection: [gettingstarted]
```
建议的默认值是“gettingstarted”，但您可能希望选择一个更适合您的特定搜索应用程序的名称。  
  
接下来，脚本会提示您分配集合的碎片数量。分片进行更详细的覆盖以后，所以如果您不确定，我们建议使用2默认，以便您可以看到集合是如何在 SolrCloud 群集中的多个节点之间分布的。  
接下来，脚本会提示您为每个分片创建副本的数量。 本指南稍后将详细介绍复制，所以如果您不确定，请使用默认值2，以便您可以看到如何在SolrCloud中处理复制。  
最后，脚本会提示您输入您的集合的配置目录名称。您可以选择_default或sample_techproducts_configs。配置目录是从server/solr/configsets/如果你愿意，您可以预先审查。_default 配置在您仍在为文档设计架构时非常有用，需要您尝试使用Solr，因为它具有无架构功能的一些flexiblity配置是非常有用的。但是，创建集合之后，可以禁用无模式功能，以便锁定模式（以便在执行此操作后编入索引的文档不会改变模式）或自行配置模式。这可以如下完成（假设你的集合名称是 mycollection）：  
curl http://host:8983/solr/mycollection/config -d '{"set-user-property": {"update.autoCreateFields":"false"}}'  
此时，您应该在本地SolrCloud群集中创建一个新的集合。要验证这一点，您可以运行“status”命令：  
```
bin/solr status
```
如果在此过程中遇到任何错误，请检查Solr的日志文件example/cloud/node1/logs和example/cloud/node2/logs。  
  
您可以通过访问Solr管理界面中的云面板来查看您的集群在集群中的部署方式：http：//localhost：8983/solr/＃/〜cloud。Solr还提供了一种使用healthcheck命令为集合执行基本诊断的方法：  
```
bin/solr healthcheck -c gettingstarted
```
healthcheck命令收集有关集合中每个副本的基本信息，例如文档数量，当前状态（活动，关闭等）和地址（副本在集群中的位置）。  
现在可以使用Post工具将文档添加到SolrCloud 。  
要在SolrCloud模式下停止Solr，您可以使用bin/solr脚本并发出stop命令，如下所示：  
```
bin/solr stop -all
```

### 从-noprompt开始

您也可以使用以下命令，以所有默认值（而不是交互式会话)开始SolrCloud：  
```
bin/solr -e cloud -noprompt
```

### 重新启动节点

您可以使用bin/solr脚本重新启动您的SolrCloud节点。例如，要重新启动在端口8983（使用嵌入式ZooKeeper服务器）上运行的node1，您应该：  
```
bin/solr restart -c -p 8983 -s example/cloud/node1/solr
```
要重新启动端口7574上运行的节点2，您可以执行以下操作：  
```
bin/solr restart -c -p 7574 -z localhost:9983 -s example/cloud/node2/solr
```
请注意，启动node2时需要指定ZooKeeper地址（-z localhost:9983），以便可以将节点1连接到群集。  

### 将节点添加到群集

将节点添加到现有集群有点高级，并且对Solr有了更多的了解。一旦使用启动脚本启动SolrCloud集群，您可以通过以下方式向其添加新节点：  
```
mkdir &lt;solr.home for new solr node&gt;
cp &lt;existing solr.xml path&gt; &lt;new solr.home&gt;
bin/solr start -cloud -s solr.home/solr -p &lt;port num&gt; -z &lt;zk hosts string&gt;
```
请注意，上述要求您创建一个Solr主目录。您需要将 solr xml 复制到 solr_home ，或者保存在ZooKeeper中的/solr.xml。  
  
将节点添加到以“bin / solr -e cloud”开头的示例的示例（使用目录结构）：  
```
mkdir -p example/cloud/node3/solr
cp server/solr/solr.xml example/cloud/node3/solr
bin/solr start -cloud -s example/cloud/node3/solr -p 8987 -z localhost:9983
```
前面的命令将在端口8987上启动另一个 solr 节点，并将 solr 主页设置为 example/cloud/node3/solr。新节点会将其日志文件写入 example/cloud/node3/logs。  
  
一旦您对 SolrCloud 示例的工作方式感到满意，我们建议使用将 Solr 带到生产中的过程，以便在生产中设置 SolrCloud 节点。  
