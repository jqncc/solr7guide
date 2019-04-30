## 升级Solr集群 
<div class="content-intro view-box ">本页介绍如何升级使用服务安装脚本安装的现有 Solr 集群。  
```
Tip：该页面上列出的步骤假定您使用默认的服务名称 solr。如果您使用备用服务名称或 Solr 安装目录，则下面提到的一些路径和命令将必须相应地进行修改。
```


## 规划升级

以下是在开始升级过程之前需要准备的事项清单：  

1 <li>检查 [Solr 版本升级说明](https://www.w3cschool.cn/solr_doc/solr_doc-us1r2fp5.html)以确定 Solr 新版本中是否有任何行为改变会影响您的安装。</li>2 <li>如果不使用复制（即 replicationFactor 小于1的集合），则应对每个集合进行备份。如果您的所有集合都使用复制，则在技术上不需要进行备份，因为您将逐个升级和验证每个节点。</li>3 <li>确定哪个 Solr 节点当前在 SolrCloud 中托管 Overseer leader 进程，因为您应该最后升级该节点。要确定监督，使用监督状态 API，请参阅：集合 API。</li>4 <li>如果可能，计划在系统维护时段内执行升级。您将会对集群（每个节点，一个接一个）执行滚动重新启动，但是我们仍然建议在系统使用率最小的时候进行升级。</li>5 <li>验证集群当前是否正常并且所有副本都处于活动状态，因为您不应该在降级的群集上执行升级。</li>6 <li>根据新的 Solr JAR 文件重新生成并测试所有自定义的服务器端组件。</li>7 <li>确定 Solr 控制脚本使用的以下变量的值：<ul><li>ZK_HOST：您当前的 SolrCloud 节点用于连接到 ZooKeeper 的 ZooKeeper 连接字符串；该值对于集群中的所有节点将是相同的。</li>8 <li>SOLR_HOST：每个 Solr 节点在加入 SolrCloud 集群时用于注册 ZooKeeper 的主机名；此值将用于在启动新的 Solr 进程时设置主机 Java系统属性。</li>9 <li>SOLR_PORT：每个 Solr 节点正在监听的端口，如 8983。</li></li>
</ol>
您现在应该准备升级您的集群。在进行生产之前，请在测试或暂存集群中验证此过程。  

## 升级过程<a href="http://lucene.apache.org/solr/guide/7_0/upgrading-a-solr-cluster.html#upgrade-process"/>

我们建议的方法是逐个升级每个 Solr 节点。换句话说，您需要停止节点，将其升级到新版本的 Solr，并在移动到下一个节点之前重新启动它。这意味着在很短的时间内，将在您的集群中运行“旧 Solr”和“新 Solr”节点。我们还假设您将把新的 Solr 节点指向您现有的 Solr 主目录，在这个目录下为节点上的每个集合管理 Lucene 索引文件。这意味着你将不需要移动任何索引文件来执行升级。
      
  

### 步骤1：停止 Solr<a href="http://lucene.apache.org/solr/guide/7_0/upgrading-a-solr-cluster.html#step-1-stop-solr"/>

从停止要升级的 Solr 节点开始。在停止节点之后，如果使用复制（即，具有 replicationFactor 小于1的集合），则验证在关闭节点上托管的所有领导者是否已经成功迁移到其他副本；您可以通过访问 Solr 管理界面中的云面板来完成此操作。如果不使用复制，那么在关闭的节点上承载的碎片的任何集合将暂时脱机。  

### 步骤2：将 Solr 作为服务安装

请按照说明将 Solr 作为服务安装在 Linux 上，记录在 Taking Solr to Production。使用该 -n 参数可避免安装程序脚本自动启动 Solr。您需要更新 /etc/default/solr.in.sh，它包含在下一步中完成升级过程的文件。  
```
Tip：如果您有一个/var/solr/solr.in.sh用于现有 Solr 安装的文件，则运行该install_solr_service.sh脚本会将该文件移动到新的位置：/etc/default/solr.in.sh。     <span style="background-color: rgb(249, 242, 244); color: rgb(199, 37, 78); white-space: nowrap;">                </span>
```


### 步骤3：设置环境变量覆盖

用文本编辑器打开 /etc/default/solr.in.sh，并验证以下变量设置是否正确，或根据需要将它们添加到包含文件的底部：  
```
ZK_HOST=SOLR_HOST=SOLR_PORT=SOLR_HOME=
```

确保您计划拥有 Solr 进程的用户是该 SOLR_HOME 目录的所有者。举例来说，如果您计划将 Solr 作为 “Solr” 用户并且 SOLR_HOME 作为 /var/solr/data，那么您需要：  
```
sudo chown -R solr: /var/solr/data
```


### 步骤4：启动 Solr<a href="http://lucene.apache.org/solr/guide/7_0/upgrading-a-solr-cluster.html#step-4-start-solr"/>

您现在准备通过执行以下操作来启动升级后的 Solr 节点：sudo service solr start。升级后的实例将加入现有集群，因为你使用的 SOLR_HOME、SOLR_PORT 以及SOLR_HOST 是由旧的 Solr 节点使用的设置；因此，新的服务器将看起来像旧节点到正在运行的集群。确保查看 /var/solr/logs/solr.log 在启动过程中记录的错误。  

### 步骤5：运行 Healthcheck<a href="http://lucene.apache.org/solr/guide/7_0/upgrading-a-solr-cluster.html#step-5-run-healthcheck"/>

在继续升级群集中的下一个节点之前，应该对已升级的节点上承载的所有集合运行 Solr healthcheck 命令。例如，如果新升级的节点承载 MyDocuments 集合的副本，则可以运行以下命令（将 ZK_HOST 替换为 ZooKeeper 连接字符串）：  
```
/opt/solr/bin/solr healthcheck -c MyDocuments -z ZK_HOST
```

查找有关该集合的任何副本的任何报告问题。  
最后，对集群中的所有节点重复步骤1-5。  
