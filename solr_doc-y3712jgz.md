## CDCR API 
<div class="content-intro view-box ">
## CDCR API

CDCR API用于控制和监视复制过程。控件操作是在集合级别执行的，即通过使用以下基本URL进行API调用：http://localhost:8983/solr/&lt;collection&gt;/cdcr。
      
  
监视器操作在核心级别执行，即通过使用以下基本URL进行API调用：http://localhost:8983/solr/&lt;core&gt;/cdcr。  
目前，没有一个CDCR API调用具有参数。  

### CDCR API入口点（控制）

    - &lt;collection&gt;/cdcr?action=STATUS：返回 CDCR 的当前状态。
    - &lt;collection&gt;/cdcr?action=START：启动CDCR复制
    - &lt;collection&gt;/cdcr?action=STOP：停止CDCR复制。
    - &lt;collection&gt;/cdcr?action=ENABLEBUFFER：启用缓冲更新。
    - &lt;collection&gt;/cdcr?action=DISABLEBUFFER：禁用更新的缓冲。

### CDCR API入口点（监控）

    - core/cdcr?action=QUEUES：获取有关每个副本的队列以及有关更新日志的统计信息。
    - core/cdcr?action=OPS：获取有关每个副本的复制性能（每秒操作）的统计信息。
    - core/cdcr?action=ERRORS：获取有关每个副本的复制错误的统计信息和其他信息。

## <span style="font-family: inherit;">CDCR</span>控制命令

### CDCR STATUS

/collection/cdcr?action=STATUS  

#### CDCR STATUS示例

示例输入：  
```
 http://host:8983/solr/&lt;collection_name&gt;/cdcr?action=STATUS
```
示例输出：  
```
{
  "responseHeader": {
  "status": 0,
  "QTime": 0
  },
  "status": {
  "process": "stopped",
  "buffer": "enabled"
  }
}
```

### ENABLEBUFFER（启用缓冲区）

/collection/cdcr?action=ENABLEBUFFER  
启用缓冲区响应表示进程的状态以及是否启用缓冲区的指示。  

#### 启用缓冲区示例

示例输入：  
```
http://host:8983/solr/&lt;collection_name&gt;/cdcr?action=ENABLEBUFFER
```
示例输出：  
```
{
  "responseHeader": {
  "status": 0,
  "QTime": 0
  },
  "status": {
  "process": "started",
  "buffer": "enabled"
  }
}
```

### DISABLEBUFFER（禁用缓冲区）

/collection/cdcr?action=DISABLEBUFFER  
禁用缓冲区响应：CDCR的状态和缓冲区被禁用的指示。  

#### 禁用缓冲区示例

示例输入：  
```
http://host:8983/solr/&lt;collection_name&gt;/cdcr?action=DISABLEBUFFER
```
示例输出：  
```
{
  "responseHeader": {
  "status": 0,
  "QTime": 0
  },
  "status": {
  "process": "started",
  "buffer": "disabled"
  }
}
```

### CDCR START

/collection/cdcr?action=START  
CDCR START响应：确认CDCR已启动，缓冲状态。  

#### CDCR START示例

示例输入：  
```
http://host:8983/solr/&lt;collection_name&gt;/cdcr?action=START
```
示例输出：  
```
{
  "responseHeader": {
  "status": 0,
  "QTime": 0
  },
  "status": {
  "process": "started",
  "buffer": "enabled"
  }
}
```

### CDCR STOP

/collection/cdcr?action=STOP  
CDCR STOP响应：CDCR的状态，包括确认CDCR已停止。  

#### CDCR STOP示例

示例输入：  
```
http://host:8983/solr/&lt;collection_name&gt;/cdcr?action=STOP
```
示例输出：  
```
{
  "responseHeader": {
  "status": 0,
  "QTime": 0
  },
  "status": {
  "process": "stopped",
  "buffer": "enabled"
  }
}
```

## CDCR监视命令

### QUEUES

/core/cdcr?action=QUEUES  

### 队列（QUEUES）响应

### <b>QUEUES输出内容</b>

输出由列表“队列（queues）”组成，其中包含（ZooKeeper）目标主机列表，它们本身包含目标集合列表。对于每个集合，都提供了队列的当前大小和最后一次成功处理的更新操作的时间戳。更新操作的时间戳是原始时间戳，即在源SolrCloud上处理此操作的时间。这允许估计复制过程的延迟。
      
  
“queues”对象还包含有关更新日志的信息，例如磁盘上的更新日志的大小（以字节为单位）（“tlogTotalSize”），事务日志文件的数量（“tlogTotalCount”）以及更新的状态日志同步器（“updateLogSynchronizer”）。  

#### QUEUES例子

示例输入：  
```
http://host:8983/solr/&lt;replica_name&gt;/cdcr?action=QUEUES
```
示例输出：  
```
{
  "responseHeader":{
    "status": 0,
    "QTime": 1
  },
  "queues":{
    "127.0.0.1: 40342/solr":{
    "Target_collection":{
        "queueSize": 104,
        "lastTimestamp": "2014-12-02T10:32:15.879Z"
      }
    }
  },
  "tlogTotalSize":3817,
  "tlogTotalCount":1,
  "updateLogSynchronizer": "stopped"
}
```

### OPS

/core/cdcr?action=OPS  

### OPS响应

输出operationsPerSecond包含一个（ZooKeeper）目标主机列表，它们本身包含一个目标集合列表。对于每个集合，都提供自复制过程开始以来每秒平均处理的操作数。这些操作进一步分为两组：添加和删除操作。
      
  

#### OPS示例

示例输入：  
```
http://host:8983/solr/&lt;collection_name&gt;/cdcr?action=OPS
```
示例输出：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":1
  },
  "operationsPerSecond":{
    "127.0.0.1: 59661/solr":{
      "Target_collection":{
          "all": 297.102944952749052,
          "adds": 297.102944952749052,
          "deletes": 0.0
      }
    }
  }
}
```

### ERRORS

/core/cdcr?action=ERRORS  

### ERRORS响应

输出由包含（ZooKeeper）目标主机列表的“错误”列表组成，这些主机本身包含目标集合列表。对于每个集合，都会提供关于在复制过程中遇到的错误的信息，例如复制器线程遇到的连续错误数，复制过程开始以来的错误请求数或内部错误数，以及最后一个错误列表遇到按时间戳排序。  

#### ERRORS示例

示例输入：  
```
http://host:8983/solr/&lt;collection_name&gt;/cdcr?action=ERRORS
```
示例输出：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":2
  },
  "errors": {
    "127.0.0.1: 36872/solr":{
      "Target_collection":{
        "consecutiveErrors":3,
        "bad_request":0,
        "internal":3,
        "last":{
          "2014-12-02T11:04:42.523Z":"internal",
          "2014-12-02T11:04:39.223Z":"internal",
          "2014-12-02T11:04:38.22Z":"internal"
        }
      }
    }
  }
}
```
  
  
