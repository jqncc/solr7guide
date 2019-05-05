## SolrCloud Autoscaling概述 
<div class="content-intro view-box ">
## SolrCloud Autoscaling概述
Solr中的自动缩放（Autoscaling）旨在提供良好的默认设置，因此SolrCloud群集在各种群集更改事件面前保持平衡和稳定。这种平衡是通过满足一系列规则和排序首选项来实现的，从而选择集群管理操作的目标。  
  
## 群集首选项
正如其名称所示，集群首选项适用于所有群集管理操作，而不管它们影响哪个集合。  
  
首选项是帮助Solr选择最大化或最小化给定度量的节点的一组条件。例如，诸如{minimize:cores}此类的首选项将帮助Solr选择节点，以使每个节点上的核心数量最小化。我们编写集群首选项的方式可以减少系统的总体负载。您可以添加多个首选项来断开关系。  
默认的集群首选项由上面的示例（{minimize : cores}）组成，这是为了最小化所有节点上的核心数量。  
您可以在“ Autoscaling群集首选项（Autoscaling Cluster Preferences）”部分了解有关首选项的更多信息。  
## 群集策略
群集策略是节点、分片或集合必须满足的一组条件，然后才能选择它作为群集管理操作的目标。无论所管理的集合是什么，这些条件都在整个群集上应用。例如，条件{"cores":"&lt;10", "node":"#ANY"}意味着任何节点总共必须少于10个Solr核心，而不管它们属于哪个集合。  
  
有很多条件可以作为基础，例如，系统负载平均值、堆使用情况、可用磁盘空间等等。支持度量的完整列表可以在描述策略属性的章节中找到。  
当节点、分片或集合不符合策略时，我们称之为违规。Solr确保集群管理操作最大限度地减少违规次数。集群管理操作目前是手动调用的。将来，这些集群管理操作可能会自动响应群集事件（例如添加或丢失的节点）而被调用。  
## 特定于集合的策略
集合可能需要除集群策略中指定的条件之外的条件。在这种情况下，我们可以创建可用于特定集合的命名策略。首先，我们可以使用set-policy API创建一个新的策略，然后指定Collection API的CREATE命令的policy=&lt;policy_name&gt;参数。  
  
```
/admin/collections?action=CREATE&amp;name=coll1&amp;numShards=1&amp;replicationFactor=2&amp;policy=policy1
```
上面的 create 集合命令将一个名为 policy1 的策略与名为 coll1 的集合相关联。只有一个策略可能与一个集合相关联。  
请注意，除了群集策略外，还应用了集合特定的策略，即它不是重写而是扩充。因此，集合将遵循集群首选项、集群策略和名为 policy1 的策略中列出的所有条件。  
您可以在“定义特定于集合的策略”部分中了解有关特定于集合的策略的更多信息。  
## 自动缩放API（Autoscaling APIs）
/admin/autoscaling中可用的自动缩放API可以用来读取和修改上面讨论的每个组件。  
您可以在Autoscaling API部分了解有关这些API的更多信息。  
