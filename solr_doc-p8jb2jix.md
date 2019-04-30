## SolrCloud自动缩放API 
<div class="content-intro view-box ">自动缩放API用于管理自动缩放策略和首选项，并获取有关群集状态的诊断信息。
      
  

## <span style="font-family: inherit;">Read </span>API

自动缩放的Read API可存放在/admin/autoscaling或/v2/cluster/autoscaling中。它返回有关配置的群集首选项，群集策略和特定于集合的策略的信息。
      
  
这个API不带任何参数。  

### <span style="font-family: inherit;">Read </span>API响应

输出将包含群集首选项，群集策略和特定于集合的策略。  

### 使用Read API的示例

示例输入：  
```
{
    "responseHeader": {
        "status": 0,
        "QTime": 2
    },
    "cluster-policy": [
        {
            "replica": "&lt;2",
            "shard": "#EACH",
            "node": "#ANY"
        }
    ],
    "WARNING": "This response format is experimental.  It is likely to change in the future."
}
```

## <span style="font-family: inherit;">Diagnostics </span>API

Diagnostics API显示群集中所有情况（如果有的话）的违规情况，以及（如果适用）集合策略。它在/admin/autoscaling/diagnostics路径上可用。  
这个API不带任何参数。  

### <span style="font-family: inherit;">Diagnostics </span>API响应

输出将包含集群中的sortedNodes节点列表，其按照总体负载按降序（由首选项确定）进行排序，并且violations是包含它们违反的条件的节点列表。  

### 使用Diagnostics API示例

这里是一个没有违规的例子，但是在这一sortedNodes节中，我们可以看到第一个节点是最负载的（根据核心数量）：  
```
{
    "responseHeader": {
        "status": 0,
        "QTime": 65
    },
    "diagnostics": {
        "sortedNodes": [
            {
                "node": "127.0.0.1:8983_solr",
                "cores": 3
            },
            {
                "node": "127.0.0.1:7574_solr",
                "cores": 2
            }
        ],
        "violations": []
    },
    "WARNING": "This response format is experimental.  It is likely to change in the future."
}
```
假设我们向群集策略添加了一个条件如下：  
```
{"replica": "&lt;2", "shard": "#EACH", "node": "#ANY"}
```
但是，由于第一个示例中的第一个节点已经有一个以上的分片副本，因此Diagnostics API将返回：  
```
{
    "responseHeader": {
        "status": 0,
        "QTime": 45
    },
    "diagnostics": {
        "sortedNodes": [
            {
                "node": "127.0.0.1:8983_solr",
                "cores": 3
            },
            {
                "node": "127.0.0.1:7574_solr",
                "cores": 2
            }
        ],
        "violations": [
            {
                "collection": "gettingstarted",
                "shard": "shard1",
                "node": "127.0.0.1:8983_solr",
                "tagKey": "127.0.0.1:8983_solr",
                "violation": {
                    "replica": "2",
                    "delta": 0
                },
                "clause": {
                    "replica": "&lt;2",
                    "shard": "#EACH",
                    "node": "#ANY",
                    "collection": "gettingstarted"
                }
            }
        ]
    },
    "WARNING": "This response format is experimental.  It is likely to change in the future."
}
```
在上面的例子中，带有端口8983的节点有两个副本shard1违反了我们的策略。  

## <span style="font-family: inherit;">Write </span>API

Write API与Read API在相同/admin/autoscaling和/v2/cluster/autoscaling端点上可用，但只能与 POST HTTP 谓词一起使用。
      
  
POST请求的有效负载是一个JSON消息，带有设置和删除组件的命令。可以在有效负载中一起指定多个命令。这些命令按照指定的顺序执行，并且这些更改是atomic的，即全部成功或者全部成功。  

### 创建和修改群集首选项

群集首选项被指定为排序首选项的列表。可以指定多个排序首选项，并按顺序应用。  
它们是使用该set-cluster-preferences命令定义的。  
每个首选项是具有以下语法的JSON映射：  
```
{'&lt;sort_order&gt;':'&lt;sort_param&gt;', 'precision':'&lt;precision_val&gt;'}
```
有关 sort_order、sort_param 和 precision 参数的允许值的详细信息，参阅 "群集首选项规范" 部分。  
群集已建好后更改群集首选项不会自动重新配置群集。但是，所有未来的群集管理操作都将使用已更改的首选项。  
示例输入：  
```
{
"set-cluster-preferences" : [
  {"minimize": "cores"}
  ]
}
```
示例输出：  
输出有一个名为"result"的键，它将根据命令是成功还是失败返回“success”或“failure”：  
```
{
    "responseHeader": {
        "status": 0,
        "QTime": 138
    },
    "result": "success",
    "WARNING": "This response format is experimental.  It is likely to change in the future."
}
```

#### 设置群集首选项示例

在这个例子中，我们添加了三个不同的参数排序的群集首选项：  
```
{
  "set-cluster-preferences": [
    {
      "minimize": "cores",
      "precision": 2
    },
    {
      "maximize": "freedisk",
      "precision": 100
    },
    {
      "minimize": "sysLoadAvg",
      "precision": 10
    }
  ]
}
```
我们可以通过将首选项设置为空列表来删除所有群集首选项。  
```
{
  "set-cluster-preferences": []
}
```

### 创建和修改群集策略

集群策略是使用该set-cluster-policy命令设置的。  
就像set-cluster-preferences，策略定义是一个定义所需的属性和值的JSON映射。  
有关策略中每个条件的允许值的详细信息，请参阅策略规范部分。  
示例输入：  
```
{
"set-cluster-policy": [
  {"replica": "&lt;2", "shard": "#EACH", "node": "#ANY"}
  ]
}
```
得到输出：  
```
{
    "responseHeader": {
        "status": 0,
        "QTime": 47
    },
    "result": "success",
    "WARNING": "This response format is experimental.  It is likely to change in the future."
}
```
通过将策略设置为空列表，我们可以删除所有群集策略条件。  
```
{
  "set-cluster-policy": []
}
```
在群集已经建立后更改群集策略不会自动重新配置群集。但是，所有未来的集群管理操作都将使用已更改的集群策略。  

### 创建和修改特定于集合的策略

该set-policy命令接受策略名称映射到该策略的条件列表。可以将多个命名策略一起指定。一个不存在的已命名的策略被创建，如果命名的策略已经被接受，那么它被替换。
      
  
有关策略中每个条件的允许值的详细信息，请参阅策略规范部分。  
输入如下：  
```
{
"set-policy": {
  "policy1": [
    {"replica": "1", "shard": "#EACH", "port": "8983"}
    ]
  }
}
```
得到输出：  
```
{
    "responseHeader": {
        "status": 0,
        "QTime": 246
    },
    "result": "success",
    "WARNING": "This response format is experimental.  It is likely to change in the future."
}
```
已经构建集合后更改策略不会自动重新配置集合。但是，所有未来的集群管理操作都将使用已更改的策略。  

### 删除特定于集合的策略

该remove-policy命令接受从Solr中删除的策略名称。被删除的策略不能附加到任何集合，否则该命令将失败。  
  
输入如下：  
```
{"remove-policy": "policy1"}
```
得到输出：  
```
{
    "responseHeader": {
        "status": 0,
        "QTime": 42
    },
    "result": "success",
    "WARNING": "This response format is experimental.  It is likely to change in the future."
}
```
如果您尝试删除集合正在使用的策略，则该命令将无法删除策略，直到集合本身被删除。  
