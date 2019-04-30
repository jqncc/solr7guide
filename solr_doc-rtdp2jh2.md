## 初始化CDCR 
<div class="content-intro view-box "><h2>CDCR初始化  
</h2>
CDCR Bootstrapping：在Solr 6.2中添加了附加功能，允许CDCR在首次启动时将整个索引从源复制到目标数据中心，作为以下过程的替代方案。对于非常大的索引，如果选择此选项，则应为初始同步分配时间。  
这是在生产环境中初始化CDCR的一般方法，它基于CDCR的初始工作安装所采取的方法，并慷慨地为阐明“现实世界”情景做出了慷慨的贡献。
      
  

    - 客户使用CDCR方法来保持远程错误恢复实例可用于生产备份。这是一个单向的解决方案。
    - 客户有26个cloud，每个cloud有2亿个资产（15GB索引）。文件总数超过48亿。源和目标cloud在2-3小时维护窗口中同步，以建立目标的基础索引。

像往常一样，从小开始是件好事。在完成其他工作之前，先同步单个cloud并监视一段时间。在找到合适的平衡之前，您可能需要多次调整您的设置。  

    - 在开始之前，停止或暂停索引器。最好在一个小维护窗口内完成。
    - 在Source停止SolrCloud实例
    - 将CDCR请求处理程序配置包括在solrconfig. xml中，如下面的示例所示：
```
&lt;requestHandler name="/cdcr" class="solr.CdcrRequestHandler"&gt;
    &lt;lst name="replica"&gt;
      &lt;str name="zkHost"&gt;${TargetZk}&lt;/str&gt;
      &lt;str name="Source"&gt;${SourceCollection}&lt;/str&gt;
      &lt;str name="Target"&gt;${TargetCollection}&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="replicator"&gt;
      &lt;str name="threadPoolSize"&gt;8&lt;/str&gt;
      &lt;str name="schedule"&gt;10&lt;/str&gt;
      &lt;str name="batchSize"&gt;2000&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="updateLogSynchronizer"&gt;
      &lt;str name="schedule"&gt;1000&lt;/str&gt;
    &lt;/lst&gt;
  &lt;/requestHandler&gt;
  &lt;updateRequestProcessorChain name="cdcr-processor-chain"&gt;
    &lt;processor class="solr.CdcrUpdateProcessorFactory" /&gt;
    &lt;processor class="solr.RunUpdateProcessorFactory" /&gt;
  &lt;/updateRequestProcessorChain&gt;
```
    
    - 将修改后的solrconfig. xml上传到在Source和Target上的ZooKeeper。
    - 将Source集合中的索引目录同步到Target集合中相应的分片节点。rsync 对此很有效。例如，如果collection1上有两个分片，每个分片有两个副本，则将相应的索引目录从以下表中复制：
        <table>
            <colgroup>
                <col/>
                    <col/>
                        <col/>
            </colgroup>
            <tbody>
                <tr>
                    <td>
                        shard1replica1Source  
                    </td>
                    <td>
                        to  
                    </td>
                    <td>
                        shard1replica1Target  
                    </td>
                </tr>
                <tr>
                    <td>
                        shard1replica2Source  
                    </td>
                    <td>
                        to  
                    </td>
                    <td>
                        shard1replica2Target  
                    </td>
                </tr>
                <tr>
                    <td>
                        shard2replica1Source  
                    </td>
                    <td>
                        to  
                    </td>
                    <td>
                        shard2replica1Target  
                    </td>
                </tr>
                <tr>
                    <td>
                        shard2replica2Source  
                    </td>
                    <td>
                        to  
                    </td>
                    <td>
                        shard2replica2Target  
                    </td>
                </tr>
            </tbody>
        </table>
    
    - 在Target（DR）端启动ZooKeeper
    - 在Target（DR）端启动SolrCloud
    - 在Source端启动ZooKeeper
    - 在Source端启动SolrCloud。作为一般规则，SolrCloud的Target（DR）端应该在Source端之前启动。
    - 使用CDCR API在Source实例上激活CDCR： 
```
http://host:port/solr/collection_name/cdcr?action=START
```
```
http://host:port/solr/&lt;collection_name&gt;/cdcr?action=START
```
    
    - Target上不需要运行/ cdcr？action = START命令
    - 禁用Target和Source上的缓冲区
```
http://host:port/solr/collection_name/cdcr?action=DISABLEBUFFER
```
    
    - 可重建的索引

