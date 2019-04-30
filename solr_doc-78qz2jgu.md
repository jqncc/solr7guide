## CDCR配置 
<div class="content-intro view-box ">
## CDCR配置<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#cdcr-configuration-2"/>

源和目标配置在数据中心处于不同集群的情况下有所不同。这里的“集群”是指单独的ZooKeeper集合控制不相交的Solr实例。这些数据中心是否在物理上是分开的，对于这个讨论来说并不重要。
      
  

### 源配置<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#source-configuration"/>

下面是一个源代码配置文件的例子，在solrconfig.xml部分。&lt;replica&gt;部分的存在导致CDCR将此群集用作源，并且不应出现在目标集合中。有关每个设置的详细信息，请见以下两个示例：  
```
&lt;requestHandler name="/cdcr" class="solr.CdcrRequestHandler"&gt;
  &lt;lst name="replica"&gt;
    &lt;str name="zkHost"&gt;10.240.18.211:2181,10.240.18.212:2181&lt;/str&gt;
    &lt;!--
    If you have chrooted your Solr information at the target you must include the chroot, for example:
    &lt;str name="zkHost"&gt;10.240.18.211:2181,10.240.18.212:2181/solr&lt;/str&gt;
    --&gt;
    &lt;str name="source"&gt;collection1&lt;/str&gt;
    &lt;str name="target"&gt;collection1&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="replicator"&gt;
    &lt;str name="threadPoolSize"&gt;8&lt;/str&gt;
    &lt;str name="schedule"&gt;1000&lt;/str&gt;
    &lt;str name="batchSize"&gt;128&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="updateLogSynchronizer"&gt;
    &lt;str name="schedule"&gt;1000&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
&lt;!-- Modify the &lt;updateLog&gt; section of your existing &lt;updateHandler&gt;
     in your config as below --&gt;
&lt;updateHandler class="solr.DirectUpdateHandler2"&gt;
  &lt;updateLog class="solr.CdcrUpdateLog"&gt;
    &lt;str name="dir"&gt;${solr.ulog.dir:}&lt;/str&gt;
    &lt;!--Any parameters from the original &lt;updateLog&gt; section --&gt;
  &lt;/updateLog&gt;
&lt;/updateHandler&gt;
```


### 目标配置<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#target-configuration"/>

下面是一个典型的目标配置。  
目标实例必须配置特定于CDCR的更新处理器链。更新处理器链必须包含CdcrUpdateProcessorFactory。此处理器的任务是确保附加到来自CDCR源SolrCloud的更新请求的版本号被重用，并且不被目标覆盖。正确配置的目标配置与此类似。  
```
&lt;requestHandler name="/cdcr" class="solr.CdcrRequestHandler"&gt;
  &lt;lst name="buffer"&gt;
    &lt;str name="defaultState"&gt;disabled&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
&lt;requestHandler name="/update" class="solr.UpdateRequestHandler"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="update.chain"&gt;cdcr-processor-chain&lt;/str&gt;
  &lt;/lst&gt;
&lt;/requestHandler&gt;
&lt;updateRequestProcessorChain name="cdcr-processor-chain"&gt;
  &lt;processor class="solr.CdcrUpdateProcessorFactory"/&gt;
  &lt;processor class="solr.RunUpdateProcessorFactory"/&gt;
&lt;/updateRequestProcessorChain&gt;
&lt;!-- Modify the &lt;updateLog&gt; section of your existing &lt;updateHandler&gt; in your
    config as below --&gt;
&lt;updateHandler class="solr.DirectUpdateHandler2"&gt;
  &lt;updateLog class="solr.CdcrUpdateLog"&gt;
    &lt;str name="dir"&gt;${solr.ulog.dir:}&lt;/str&gt;
    &lt;!--Any parameters from the original &lt;updateLog&gt; section --&gt;
  &lt;/updateLog&gt;
&lt;/updateHandler&gt;
```


### 配置详情<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#configuration-details"/>

配置细节，默认和选项如下：  

#### 副本元素<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#the-replica-element"/>

可以将CDCR配置为将更新请求转发到一个或多个目标集合。目标集合是通过“副本”列表定义的，如下所示：  
- zkHost  

    
        目标SolrCloud的ZooKeeper的主机地址。通常这是Target ZooKeeper集合中每个节点的逗号分隔列表。该参数是必需的。  
    
- Source  

    
        要复制的源SolrCloud上集合的名称。该参数是必需的。  
    
- Target  

   
        要将更新转发到的Target SolrCloud上的集合的名称。该参数是必需的。  
    


#### <span style="font-family: inherit;">Replicator</span>元素<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#the-replicator-element"/>

CDC Replicator是负责将更新转发到副本的组件。Replicator将监视Source集合的更新日志，并将任何新的更新转发到Target集合。
      
  
Replicator使用固定的线程池并行地将更新转发到多个副本。如果配置了多个副本，则一个线程将以循环方式每次从一个副本转发一批更新。Replicator可以使用“复制器”列表进行配置，如下所示：  
- threadPoolSize  

    
        用于转发更新的线程数。建议每个副本一个线程。默认是<code>2</code>。  
    
- schedule  

    
        监视更新日志的延迟时间（以毫秒为单位）。默认是<code>10</code>。  
    
- batchSize  

    
        在一个批处理中发送的更新数。最佳大小取决于文档的大小。大批量的大文件可能会显着增加您的内存使用量。默认是<code>128</code>。  
    


#### updateLogSynchronizer元素<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#the-updatelogsynchronizer-element"/>

Expert：非leader节点需要不时地将其更新日志与其leader节点同步，以清除弃用的事务日志文件。默认情况下，每分钟执行一次这样的同步过程。可以使用“updateLogSynchronizer”列表修改同步计划，如下所示：
      
  
- schedule  

    
        同步更新日志的延迟时间（以毫秒为单位）。默认是<code>60000</code>。  
    


#### <span style="font-family: inherit;">Buffer</span>元素<a href="http://lucene.apache.org/solr/guide/7_0/cross-data-center-replication-cdcr.html#the-buffer-element"/>

缓冲更新时，更新日志将无限期地存储所有更新。建议在正常操作期间禁用源和目标群集上的缓冲，因为启用缓冲时，更新日志将无限制地增长。保持启用缓冲用于特殊维护期间。启动时可以使用“缓冲区”列表和参数“defaultState”禁用缓冲区，如下所示：  
- defaultState  

    
        启动时缓冲区的状态。默认是<code>enabled</code>。  
    

<h3>
                缓冲仅适用于维护窗口
                
                    </h3>
缓冲旨在增加维护窗口。应该牢记以下几点：  

    - 
        启用缓冲时，更新日志将无限制地增长; 他们将永远不会被清除。  
    
    - 
        在正常操作期间，如果目标数据中心不可用，则更新日志将在源数据中心自动生成；没有必要为CDCR启用缓冲来处理常规的网络中断。  
        <ul>
            <li>
                因此，建议在源数据中心监视磁盘使用情况，作为目标数据中心正在接收更新的额外检查。  
            
        
    </li>
    <li>
        不应在目标数据中心启用缓冲, 因为更新日志将不受限制地累积。
              
          
    </li>
    <li>
        如果启用了缓冲功能，那么禁用更新日志时，将更新日志的内容发送到目标数据中心。这个过程可能需要一些时间。  
        
            - 
                在将新的更新发送到源数据中心之前，不会触发更新日志清理。  
            
        
    </li>
</ul>
