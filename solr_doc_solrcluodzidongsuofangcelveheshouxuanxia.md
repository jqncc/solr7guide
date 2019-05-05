## SolrCluod自动缩放策略和首选项 
自动缩放策略和首选项是一组规则和排序首选项，帮助Solr选择群集管理操作的目标，以便集群上的总体负载保持平衡。
      
  

## SolrCloud群集首选项规范

首选项是向Solr提示如何根据节点的使用情况对节点进行排序。默认的群集首选项是按节点所承载的 Solr 核心（或副本）的总数进行排序。因此，默认情况下，在选择要添加副本的节点时，Solr可以应用这个首选项并选择核心数量最少的节点。
      
  
可以添加多个首选项来断开连接。例如，如果两个节点上的内核数量相同，我们可以选择使用空闲磁盘空间来断开连接，因此可以选择空闲磁盘空间较高的节点作为集群操作的目标。  
每个首选项的形式如下：  
```
{"&lt;sort_order&gt;":"&lt;sort_param&gt;", "precision":"&lt;precision_val&gt;"}
```
- sort_order  
   
        该值可以是<code>maximize</code>或者<code>minimize</code>。<code>minimize</code>将最小值的节点排序为最小负载。例如，<code>{"minimize":"cores"}</code>将核心数量最少的节点排序为负载最少的节点。排序顺序，例如<code>{"maximize":"freedisk"}</code>将具有最大可用磁盘空间的节点排序为负载最少的节点。  
        
            系统的目标是使每个节点的负载最小。所以，在一个<code>MOVEREPLICA</code>操作的情况下，它通常以最负载的节点为目标，并将其卸载。在一种更加负载较少的加载，<code>minimize</code>类似于降序排序，<code>maximize</code>类似于按升序排序。  
          
        
            这是一个必需的参数。  
          
    
- sort_param  
    
        必须指定以下唯一支持的参数之一：  
        
            <ol>
                <li>
                    <code>cores</code>：节点上的Solr核心总数。  
                
                - 
                    <code>freedisk</code>：Solr数据主目录的可用磁盘空间量。这总是以千兆字节为单位。  
                
                - 
                    <code>sysLoadAvg</code>：由Metrics API在<code>solr.jvm/os.systemLoadAverage</code>密钥下报告的节点上的系统负载平均值。这通常是介于0和1之间的double值，值越高，节点的加载越多。  
                
                - 
                    <code>heapUsage</code>：由Metrics API在<code>solr.jvm/memory.heap.usage</code>密钥下报告的节点的堆使用情况。这通常是介于0和1之间的double值，值越高，节点的加载越多。  
                
            
          
    </li><li>precision  
    
        precision告诉系统两个值之间的最小（绝对）差值，将它们视为不同的值。  
        
            例如，10的精度<code>freedisk</code>意味着两个节点的空闲磁盘空间相互在10GB以内，为了排序的目的应该被视为相等。这有助于创建关系，没有它指定多个首选项是没有用的。这是一个可选参数，其值必须是正整数。最大值<code>precision</code>必须小于最大值<code>sort_value</code>，如果有的话。  
          
    </li>
</ul>
有关如何管理群集首选项的详细信息，请参阅set-cluster-preferences API部分。  

### 群集首选项的示例

#### 默认首选项

以下显示了默认群集首选项。当没有使用Autoscaling API设置明确的群集首选项时，这由Solr 自动应用。  
```
[
  {"minimize":"cores"}
]
```

#### 最小化核心; 最大化可用磁盘

在这个例子中，我们希望最小化Solr内核的数量，并且在一个连接的情况下，最大化每个节点上的可用磁盘空间量。  
```
[
  {"minimize" : "cores"},
  {"maximize" : "freedisk"}
]
```

#### 将精度添加到可用磁盘; 最小化系统负载

在这个例子中，我们给freedisk参数添加了一个精度，这样相互之间的可用磁盘空间在10GB以内的节点被认为是相等的。在这种情况下，通过最小化sysLoadAvg来打破联系。  
```
[
  {"minimize" : "cores"},
  {"maximize" : "freedisk", "precision" : 10},
  {"minimize" : "sysLoadAvg"}
]
```

## 策略规范

策略是每个节点都要满足的硬性规则。如果一个节点不满足规则，那么它被称为违规。Solr确保在调用任何集群管理操作时，将违规数量降至最低。  

### 策略属性

策略可以具有以下属性：  
- cores  
  
        这是一个适用于整个群集的特殊属性。它只能与<code>node</code>属性一起使用，而不能与其他属性一起使用。该属性是可选的。  
    
- collection  
    
        策略规则应该应用到的集合的名称。如果省略，则该规则适用于所有集合。该属性是可选的。  
    
- shard  
    
        策略规则应该应用到的分片的名称。如果省略，则该规则适用于集合中的所有分片。它支持一个特殊的值<code>#EACH</code>，这意味着该规则适用于集合中的每个分片。  
    
- replica  
    
        为满足规则而必须存在的副本数量。这必须是一个正整数。这是一个必需的属性。  
    
- strict  
    
        一个可选的布尔值。默认是<code>true</code>。如果 true，则必须满足该规则。如果 false，Solr尽力满足规则，但是如果没有节点可以满足规则，那么可以选择任何节点。  
    

除了以上属性之外，还可以指定下列属性中的一个属性：  
- node  
    
        规则应该适用的节点的名称。默认值是<code>#ANY</code>这意味着群集中的任何节点都可以满足规则。  
    
- port  
   
        规则应该适用的节点的端口。  
    
- freedisk  
    
        以千兆字节为单位的可用磁盘空间。这必须是一个正的64位整数值。  
    
- host  
   
        节点的主机名。  
    
- sysLoadAvg  
   
        由Metrics API在<code>solr.jvm/os.systemLoadAverage</code>密钥下报告的节点的系统负载平均值这是0到1之间的浮点值。  
    
- heapUsage  
    
        由Metrics API在<code>solr.jvm/memory.heap.usage</code>密钥下报告的节点的堆使用情况这是0到1之间的浮点值。  
    
- nodeRole  
    
        节点的作用。目前唯一支持的值是<code>overseer</code>。  
    
- ip_1 , ip_2, ip_3, ip_4  
    
        对IP地址最重要的最重要的部分。例如，对于一个IP地址<code>192.168.1.2</code>，<code>ip_1 = 2</code>，<code>ip_2 = 1</code>，<code>ip_3 = 168</code>，<code>ip_4 = 192</code>。  
    
- sysprop.&lt;system_property_name&gt;  
    
        启动时在节点上设置的任何系统属性。  
    

### 策略运算符

策略中的每个属性都可以指定下列运算符之一以及该值。
      
  

    - &lt;： 少于
    - &gt;： 大于
    - !：不等于
    - None意思是平等的

### 策略规则的例子

#### 限制副本放置

不要在同一个节点上放置同一个分片的多个副本：  
```
{"replica": "&lt;2", "shard": "#EACH", "node": "#ANY"}
```

#### 限制每个节点的内核

不要在任何节点放置10个以上的内核。此规则只能添加到群集策略中，因为它提到了仅适用于群集范围的cores属性。  
```
{"cores": "&lt;10", "node": "#ANY"}
```

#### 基于端口的本地副本

在端口上运行的节点上正好放置每个xyz集合分片的一个副本8983  
```
{"replica": 1, "shard": "#EACH", "collection": "xyz", "port": "8983"}
```

#### 基于系统属性放置副本

将所有副本放在具有availability_zone=us-east-1a系统属性的节点上。请注意，我们必须在负面意义上编写此规则，即0个副本必须位于不具有availability_zone=us-east-1a系统属性的节点上：  
```
{"replica": 0, "sysprop.availability_zone": "!us-east-1a"}
```

#### 基于节点角色放置副本

不要在具有overseer角色的节点上放置任何副本。请注意，该角色是由addRole集合API 添加的。它不是自动成为当前监督者的节点。  
```
{"replica": 0, "nodeRole": "overseer"}
```

#### 基于可用磁盘的地方副本

将所有复制副本放在空闲大于500GB的节点中。这里又一次，我们必须在负面意义上写下规则。  
```
{"replica": 0, "freedisk": "&lt;500"}
```

#### 尝试基于可用磁盘放置副本

尽可能将所有副本放在空闲大于500GB的节点中。在这里，我们使用严格的关键字来表明这个规则是尽最大努力的。  
```
{"replica": 0, "freedisk": "&lt;500", "strict" : false}
```

## 定义特定于集合的策略

默认情况下，群集策略（如果存在）将自动用于群集中的所有集合。但是，我们可以通过指定策略名称和policy参数来创建可在创建时附加到集合的命名策略。
      
  
使用特定于集合的策略时，该策略中的规则将附加到群集策略中的规则中，并使用这两者的组合。因此，建议不要将规则添加到与群集策略中的集合特定的策略中。这样做会使群集中的所有节点都不符合所有标准，从而使策略失效。  
可以使用特定于集合的策略来覆盖群集策略中指定的条件。例如，如果某个子句{replica:'&lt;3', node:'#ANY'}出现在群集策略中，并且特定于集合的策略中有一个子句{replica:'&lt;4', node:'#ANY'}，则群集策略将被忽略，以支持集合策略。  
另外，如果在创建集合时指定了 maxShardsPerNode，则必须满足 maxShardsPerNode 和策略规则。  
某些属性，如cores，只能在集群策略中使用。有关详细信息，请参阅上面的策略属性部分。  
该策略由这些Collections API命令使用：  

    - CREATE
          
    
    - CREATESHARD
    - ADDREPLICA
    - RESTORE
          
    
    - SPLITSHARD

将来，Autoscaling框架将使用策略和首选项来自动更改群集，以响应诸如正在添加或丢失的节点之类的事件。  
