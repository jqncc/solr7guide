## 使用ZooKeeper管理配置文件 
<div class="content-intro view-box ">使用SolrCloud，您的配置文件保存在ZooKeeper中。  
  
在以下任何一种情况下都会上传这些文件：  
- 当您使用bin/solr脚本启动SolrCloud示例时。
- 当您使用bin/solr脚本创建一个集合时。
- 显式上传配置集到ZooKeeper时。
## 启动引导
当您第一次使用bin/solr -e cloud尝试 SolrCloud 时，相关的配置集会自动上传到ZooKeeper，并与新创建的集合链接。  
下面的命令将启动SolrCloud，默认的集合名称（gettingstarted）和默认的configset（_default）被上传并链接到它。  
```
bin/solr -e cloud -noprompt
```
您也可以在使用带有-d选项的bin/solr脚本创建集合时明确上载配置目录，例如：  
```
bin/solr create -c mycollection -d _default
```
create命令会将_default配置目录的一个副本上传到/configs/mycollection下的ZooKeeper。有关创建集合的create命令的更多详细信息，请参阅[Solr控制脚本参考](https://www.w3cschool.cn/solr_doc/solr_doc-m13y2fs3.html)页面。  
  
一旦配置目录已经上传到ZooKeeper，您可以使用Solr控制脚本来更新它们。  
提示：最好将这些文件保存在版本控制之下。  
## 使用bin / solr或SolrJ上传配置文件
在生产环境中，可以使用Solr的Solr控制脚本或CloudSolrClient.uploadConfig java方法将配置集上传到独立于集合创建的ZooKeeper上。  
  
以下命令可用于使用bin/solr脚本上传新的configset。  
```
bin/solr zk upconfig -n &lt;name for configset&gt; -d &lt;path to directory with configset&gt;
```
强烈建议将配置保存在版本控制系统中，例如：Git、SVN或类似的软件中。  
## 管理您的SolrCloud配置文件
要更新或更改您的SolrCloud配置文件：  
1 <li>使用源代码管理签出过程从ZooKeeper下载最新的配置文件。</li>2 <li>进行更改。</li>3 <li>将更改的文件提交到源代码管理。</li>4 <li>将更改推回到ZooKeeper。</li>5 <li>重新加载集合，以使更改生效。</li>
## 在第一次群集启动之前准备ZooKeeper
如果您将与其他应用程序共享相同的ZooKeeper实例，则应在ZooKeeper中使用chroot。请参阅ZooKeeper chroot的说明。  
  
有某些配置文件包含群集范围的配置。由于其中一些对群集正常运行至关重要，因此您可能需要在启动Solr群集之前首先将这些文件上传到ZooKeeper。这样的配置文件的例子（不详尽）是solr.xml，security.json和clusterprops.json。  
例如，如果您想在ZooKeeper保留solr.xml，以避免将其复制到每个节点的solr_home目录，可以使用bin/solr实用程序（Unix示例）将其推送到ZooKeeper：  
```
bin/solr zk cp file:local/file/path/to/solr.xml zk:/solr.xml -z localhost:2181
```
