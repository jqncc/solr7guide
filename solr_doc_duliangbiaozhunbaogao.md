## Solr度量标准报告 
<div class="content-intro view-box ">Solr包含一个开发人员API和工具，用于在Solr服务及其各个组件的整个生命周期中收集详细的以性能为导向的度量标准。
      
  
在内部，此功能使用 Dropwizard 度量 API，该API使用以下类别的度量工具来度量事件：  

    - 计数器（counters）- 只需计数事件。它们提供一个单一的长期值，例如请求的数量。
    - 表（meters）- 另外计算事件的速率。提供一个计数（如上所述）以及1、5和15分钟的指数衰减速率，类似于Unix系统的负载平均值。
    - 直方图（histograms）- 根据它们的值计算事件的近似分布。提供以下近似的统计数据，如上述类似的指数衰减：平均值（算术平均数）、中位数、最大值、最小值、标准偏差和75 个，95 个，98 个，99 个和999 个百分位数。
    - 定时器（timers）- 测量事件的数量和持续时间。它们提供计时的计数和直方图。
    - 仪表（gauges）- 提供当前值的即时读取，例如当前队列深度、当前活动连接数量，可用堆大小。

每个具有唯一名称的相关度量标准都在度量注册表中进行管理。Solr维护了几个这样的注册表，每个都对应一个高级组，例如：jvm、jetty、node和core（参见下面的公制注册表）。
      
  
对于每个组（或每个注册表），可以有几个reporter，这些reporter是负责从选定的注册中心到外部系统的度量的通信的组件。目前实施的reporter支持通过JMX、Ganglia、Graphite和SLF4J发布指标。  
还有一个专门的/admin/metrics处理程序，可以查询以报告所有或一个子集的当前指标从多个注册。  

## 度量册表

Solr包含多个度量注册表，它将相关的度量标准进行分组。
      
  
度量标准从流程开始直到关闭时，通过组件的所有生命周期进行维护和累积 - 例如，通过可能的几个加载，卸载或重命名操作来跟踪特定SolrCore的度量标准，并且仅在核心被明确删除。但是，度量标准在进程重新启动之间不会持久；重新启动Solr将放弃所有收集的度量值。  
以下这些是收集的主要度量标准组：  

### JVM注册表

此注册表在solr.jvm中返回，并包含以下信息。使用Metrics API发出请求时，您可以指定&amp;group=jvm限制为仅限这些度量标准。
      
  

    - 直接和映射的缓冲池
    - 类加载/卸载
    - 操作系统内存、CPU时间、文件描述符、交换、系统负载
    - GC计数和时间
    - 堆，非堆内存和GC池
    - 线程数量、状态和死锁
    - 系统属性，如Java信息、各种安装目录路径、端口和类似信息。您可以通过修改solr.xml来控制这里显示的内容。

### <span style="font-family: inherit;">Node / CoreContainer</span>注册表

此注册表在solr.node返回并包含以下信息。使用Metrics API发出请求时，您可以指定&amp;group=node限制为仅限这些度量标准。
      
  

    - 处理程序请求（计数，计时）：集合、信息、管理、configSets等
    - 内核的数量（已加载、惰性、已卸载）

### Core（SolrCore）注册表

Core (SolrCore) 注册表包括solr.core.&lt;collection&gt;，每个核心一个。使用Metrics API发出请求时，您可以指定&amp;group=core限制为仅限这些度量标准。  

    - 所有常见的RequestHandler-s报告：请求计时器/计数器、超时、错误。
    - 索引级事件：次要/主要合并的计量单位，已合并文档的数量，已删除文档的数量，当前正在运行的合并的测量器及其大小。
    - 分片复制和事务日志重放（TBD，SOLR-9856）
    - 分片处理程序和更新处理程序的打开/可用/挂起连接

### <span style="font-family: inherit;">Jetty</span>注册表

此注册表返回solr.jetty并包含以下信息。使用Metrics API发出请求时，您可以指定&amp;group=jetty限制为仅限这些度量标准。
      
  

    - 线程和池
    - 连接和请求计时器
    - HTTP类（1xx，2xx等）响应的表

未来，将为分片领导者和集群节点添加度量，包括来自每个核心度量的聚合。  

## 度量配置

系统中可用的指标可以在solr.xml通过修改&lt;metrics&gt;元素来定制。  
```
Note：有关该Solr.xml文件的更多信息，在何处查找以及如何编辑， 请参阅“Solr.xml的格式”部分。
```

### &lt;metrics&gt; &lt;hiddenSysProps&gt;元素

本部分的solr.xml允许您定义被认为系统敏感的系统属性，但是不应通过Metrics API公开。
      
  
如果未定义此部分，将使用以下默认配置来隐藏密码和身份验证信息：  
```
&lt;metrics&gt;
  &lt;hiddenSysProps&gt;
    &lt;str&gt;javax.net.ssl.keyStorePassword&lt;/str&gt;
    &lt;str&gt;javax.net.ssl.trustStorePassword&lt;/str&gt;
    &lt;str&gt;basicauth&lt;/str&gt;
    &lt;str&gt;zkDigestPassword&lt;/str&gt;
    &lt;str&gt;zkDigestReadonlyPassword&lt;/str&gt;
  &lt;/hiddenSysProps&gt;
&lt;/metrics&gt;
```

### &lt;metrics&gt; &lt;reporter&gt;元素

reporter使用Solr生成的度量数据。有关如何配置自定义reporter的详细信息, 请参阅下面的reporter部分。
      
  

### &lt;metrics&gt; &lt;suppliers&gt;元素

供应商帮助Solr生成度量数据。solr.xml的&lt;metrics&gt;&lt;suppliers&gt;部分允许您定义自己的度量标准实现并为其配置参数。  
自定义度量标准供应商的实现超出了本指南的范围，但通过下面描述的元素，还可以使用默认实现进行其他自定义设置。  
此元素定义计数器供应商的实现和配置。默认实现不支持任何配置。  
- &lt;counter&gt;  
    
        该元素定义了<code>Counter</code>供应商的实现和配置。默认实现不支持任何配置。  
    
- &lt;meter&gt;  
   
        这个元素定义了一个<code>Meter</code>供应商的实现。默认的实现支持一个额外的参数：  
        
            <ul><li>&lt;str name="clock"&gt; 参数  
              
                    用于计算EWMA速率的时钟类型。支持的值是：  
                    
                        <ul>
                            <li>
                                <code>user</code>，默认值，使用<code>System.nanoTime()</code>
                                  
                            
                            - 
                                <code>cpu</code>，它使用当前线程的CPU时间  
                            
                        
                      
                </li>
            </ul>
          
     </li><li>&lt;histogram&gt;  
   
        这个元素定义了一个<code>Histogram</code>供应商的实现。这个元素也支持上面用<code>meter</code>元素显示的 clock 参数，还有：  
        
            - &lt;str name="reservoir"&gt;   
                
                    要使用的 Reservoir 实现的完全限定的类名称。默认值是<code>com.codahale.metrics.ExponentiallyDecayingReservoir</code>，但是Solr使用的Codahale Metrics库还有其他选项。在上述提到的限制内支持以下参数：  
                    
                        <ul>
                            <li>
                                <code>size</code>，存储库大小。默认值是1028。  
                            
                            - 
                                <code>alpha</code>，衰变参数。默认值是0.015。这只适用于<code>ExponentiallyDecayingReservoir</code>。  
                            
                            - 
                                <code>window</code>，窗口的大小，以秒为单位，只对<code>SlidingTimeWindowReservoir</code>有效。默认值是300（5分钟）。  
                            
                        
                      
                </li>
            </ul>
          
     </li><li>&lt;timer&gt;
    
        这个元素定义了一个<code>Timer</code>供应商的实现。默认实现支持上面描述的<code>clock</code>和<code>reservoir</code>参数。  
    </li>
</ul>
作为solr.xml其中的一部分定义，它定义了这些自定义参数中的一些，下面定义了默认的Meter表供应商具有非默认的clock，默认Timer值与非默认的库一起使用：  
```
&lt;metrics&gt;
  &lt;suppliers&gt;
    &lt;meter&gt;
      &lt;str name="clock"&gt;cpu&lt;/str&gt;
    &lt;/meter&gt;
    &lt;timer&gt;
      &lt;str name="reservoir"&gt;com.codahale.metrics.SlidingTimeWindowReservoir&lt;/str&gt;
      &lt;long name="window"&gt;600&lt;/long&gt;
    &lt;/timer&gt;
  &lt;/suppliers&gt;
&lt;/metrics&gt;
```

## reporter

reporter配置在&lt;metrics&gt;&lt;reporter&gt;部分的solr.xml文件中指定，例如：  
```
&lt;solr&gt;
 &lt;metrics&gt;
  &lt;reporter name="graphite" group="node, jvm" class="org.apache.solr.metrics.reporters.SolrGraphiteReporter"&gt;
    &lt;str name="host"&gt;graphite-server&lt;/str&gt;
    &lt;int name="port"&gt;9999&lt;/int&gt;
    &lt;int name="period"&gt;60&lt;/int&gt;
  &lt;/reporter&gt;
  &lt;reporter name="collection1Updates" registry="solr.core.collection1" class="org.apache.solr.metrics.reporters.SolrSlf4jReporter"&gt;
    &lt;int name="period"&gt;300&lt;/int&gt;
    &lt;str name="prefix"&gt;example&lt;/str&gt;
    &lt;str name="logger"&gt;updatesLogger&lt;/str&gt;
    &lt;str name="filter"&gt;QUERYHANDLER./update&lt;/str&gt;
  &lt;/reporter&gt;
 &lt;/metrics&gt;
...
&lt;/solr&gt;
```
这个例子配置了两个reporter：Graphite和SLF4J。请参阅下面有关如何配置reporter的更多细节。  

### reporter参数

reporter插件使用以下参数：  

    - name - （必填）reporter插件的唯一名称。
    - class -（必需的）完全合格的插件实现类，必须扩展SolrMetricReporter
    - group - （可选）一个或多个预定义的组（请参阅上文）
    - registry - （可选）一个或多个有效的完全限定的注册表名称
    - 如果指定了group和registry属性，则只考虑group属性。如果这两个属性都没有指定，那么插件将被用于所有组和注册表。可以指定多个组或注册表名称，用逗号或空格分隔。

此外，可以在嵌套元素中指定几个特定于实现的初始化参数。SLF4J，Ganglia和Graphite reporter有一些共同点：  

    - period - （可选int）报告之间的时间间隔（秒）。默认值是60。
    - prefix - （可选str）要添加到度量标准名称中的前缀，可能对相关Solr实例的逻辑分组有帮助，例如计算机名称或集群名称。默认是空字符串，即，只需使用注册表名称和度量标准名称即可形成完全限定的度量标准名称。
    - filter - （可选str）如果不为空，那么只有以此值开头的度量名称才会被报告。默认是没有过滤，即，来自选定注册表的所有度量都将被报告。

reporter被实例化为他们被配置的每个组和注册表，当相应的组件被初始化时（例如，在JVM启动或SolrCore加载）。
      
  
当reporter被创建时，他们的配置被验证（例如，建立必要的连接）。在初始化阶段未捕获的错误导致reporter从运行配置中被丢弃。  
当相应组件关闭时（例如，在SolrCore关闭或JVM关闭时），reporter被关闭，但是他们报告的度量标准仍然保留在相应的注册表中，如上一节所述。  
以下各节提供了有关实现特定参数的信息。所有与Solr一起提供的实现类可以在org.apache.solr.metrics.reporters下面找到。  

### JMX Reporter

JMX Reporter使用这个org.apache.solr.metrics.reporters.SolrJmxReporter类。  
它需要以下参数：  

    - domain- （可选字符串）JMX域名。如果未指定，则使用注册表名称。
    - serviceUrl - （可选字符串）JMX服务器的服务URL。如果未指定，Solr将尝试发现JVM是否具有MBean服务器并将使用该地址。有关更多信息，请参阅下文。
    - agentId - （可选字符串）JMX服务器的代理ID。请注意，serviceUrl或者agentId可以指定，但不能同时指定 - 如果同时指定了那么将使用默认的MBean服务器。

reporter创建的对象名称是分层的、点分隔的，并且结构正确，以形成相应的层次结构，例如JConsole。该层次结构由以下元素按照自顶向下的顺序组成：  

    - 注册表名称（例如，solr.core.collection1.shard1.replica1）。由点分隔的注册表名称也被分割成ObjectName层次结构级别，因此该注册表的度量将显示在JConsole下的/solr/core/collection1/shard1/replica1中，每个域部分都被分配给dom1, dom2, …​ domN属性。
    - reporter 名称（reporter的name属性值）
    - 请求处理程序的类别，范围和名称
    - 或来自其他组件的度量的其他的name1, name2, …​ nameN元素。

JMX Reporter取代了7.0之前的Solr版本中可用的JMX功能。如果您从早期版本升级并在Solr启动时运行MBean Server，则Solr将自动发现本地MBean服务器的位置，并使用SolrJmxReporter的默认配置。  
您可以在启动时通过将-Dcom.sun.management.jmxremote添加到启动命令来启动具有系统属性的本地 MBean 服务器到启动命令。这不会将reporter配置添加到solr.xml中，所以如果您使用系统属性启用它，则必须始终使用系统属性启动Solr，否则在后续启动时将不启用 JMX。  

### SLF4J Reporter

SLF4J Reporter使用这个org.apache.solr.metrics.reporters.SolrSlf4jReporter类。  
除了上面的公共参数外，它还需要下列参数：
      
  

    - logger - （可选的str）要使用的logger的名称。默认值为空，在这种情况下，如果在插件配置中指定，则将使用组或注册表名称。

用户可以指定logger名称（例如，Log4j 配置中相应的日志记录配置），以便将与度量相关的日志记录输出到单独的文件，然后由外部应用程序进行处理。  
本reporter生成的每个日志行由特定于配置的字段和一条符合以下格式的消息组成：  
```
type=COUNTER, name={}, count={}
type=GAUGE, name={}, value={}
type=TIMER, name={}, count={}, min={}, max={}, mean={}, stddev={}, median={}, p75={}, p95={}, p98={}, p99={}, p999={}, mean_rate={}, m1={}, m5={}, m15={}, rate_unit={}, duration_unit={}
type=METER, name={}, count={}, mean_rate={}, m1={}, m5={}, m15={}, rate_unit={}
type=HISTOGRAM, name={}, count={}, min={}, max={}, mean={}, stddev={}, median={}, p75={}, p95={}, p98={}, p99={}, p999={}
```
（花括号仅作为实际值的占位符添加）。  

### Graphite Reporter

该Graphite Reporter使用org.apache.solr.metrics.reporters.SolrGraphiteReporter）类。  
除了上面的通用属性之外，它还具有以下属性：  

    - host - （必需的 str）主机名，其中正在运行Graphite服务器。
    - port - （必需的 int）服务器的端口号
    - pickled - （可选的 bool）使用“pickled”Graphite协议，可能会更有效。默认值为 false（使用纯文本协议）。

如果使用纯文本协议（pickled==false），则可以使用reporter与Graphite以外的其他系统进行集成，只要它们能够接受以以下格式的空间分隔和以线为导向的网络输入：  
```
dot.separated.metric.name[.and.attribute] value epochTimestamp
```
例如：  
```
example.solr.node.cores.lazy 0 1482932097
example.solr.node.cores.loaded 1 1482932097
example.solr.jetty.org.eclipse.jetty.server.handler.DefaultHandler.2xx-responses.count 21 1482932097
example.solr.jetty.org.eclipse.jetty.server.handler.DefaultHandler.2xx-responses.m1_rate 2.5474287707930614 1482932097
example.solr.jetty.org.eclipse.jetty.server.handler.DefaultHandler.2xx-responses.m5_rate 3.8003171557510305 1482932097
example.solr.jetty.org.eclipse.jetty.server.handler.DefaultHandler.2xx-responses.m15_rate 4.0623076220244245 1482932097
example.solr.jetty.org.eclipse.jetty.server.handler.DefaultHandler.2xx-responses.mean_rate 0.5698031798408144 1482932097
```

### Ganglia Reporter

该Ganglia Reporter使用org.apache.solr.metrics.reporters.SolrGangliaReporter类。
      
  
除了上面的常见参数外，它还有以下参数：  

    - host - （必需的 str）主机名Ganglia服务器在哪里运行。
    - port - （必需的 int）服务器的端口号
    - multicast - （可选 bool）当真正使用多播UDP通信时，否则使用UDP单播。默认值为 false。

## 核心级度量

这些度量标准仅在每个核心的基础上可用。跨核心汇总的度量尚不可用。  

### 索引合并度量标准

这些度量标准在每个核心（例如 solr.core.collection1…​.）的相应注册表中收集，在INDEX类别下。
      
  
基本度量标准总是被收集 - 在solrconfig.xml的/config/indexConfig/metrics部分中使用布尔参数可以打开其他度量的集合：  
```
&lt;config&gt;
  ...
  &lt;indexConfig&gt;
    &lt;metrics&gt;
      &lt;majorMergeDocs&gt;524288&lt;/majorMergeDocs&gt;
      &lt;bool name="mergeDetails"&gt;true&lt;/bool&gt;
      &lt;bool name="directoryDetails"&gt;true&lt;/bool&gt;
    &lt;/metrics&gt;
    ...
  &lt;/indexConfig&gt;
...
&lt;/config&gt;
```
收集以下指标：  

    - INDEX.merge.major - 至少包含“majorMergeDocs”的合并操作的计时器（此参数的默认值为512k文档）。
    - INDEX.merge.minor - 包含少于“majorMergeDocs”的合并操作的计时器。
    - INDEX.merge.errors - 合并错误的计数器。
    - INDEX.flush - meter索引刷新操作。

此外，还会报告以下量规，以帮助监视索引合并操作的瞬间状态：  

    - INDEX.merge.major.running- 正在运行的主要合并操作的数量（取决于所使用的 MergeScheduler 的实现，可以同时运行多个合并操作）。
    - INDEX.merge.minor.running - 如上所述，对于小规模的合并操作。
    - INDEX.merge.major.running.docs - 当前在主要合并操作中合并的段中的文档总数。
    - INDEX.merge.minor.running.docs - 如上所述，对于小规模的合并操作。
    - INDEX.merge.major.running.segments - 当前在主要合并操作中合并的段数。
    - INDEX.merge.minor.running.segments - 如上所述，对于小规模的合并操作。

如果布尔标志mergeDetails为true，则收集以下附加度量标准：  

    - INDEX.merge.major.docs - 主要合并操作中合并的文档的数量。
    - INDEX.merge.major.deletedDocs - 主要合并操作中删除的文档数量的表

## Metrics API

该admin/metrics端点提供对所有度量组的所有度量的访问。
      
  
有几个查询参数可用于将您的请求仅限于某些度量：  
- group  

        要检索的度量标准组。默认值是<code>all</code>检索所有组的所有度量标准。其他可能的值是：<code>jvm</code>，<code>jetty</code>，<code>node</code>和<code>core</code>。请求中可以指定多个组；多个组名应该用逗号分隔。  
    
- type  

        要检索的度量的类型。默认是<code>all</code>检索所有度量类型。其他可能的值是<code>counter</code>，<code>gauge</code>，<code>histogram</code>，<code>meter</code>和<code>timer</code>。一个请求中可以指定多个类型；多个类型应该用逗号隔开。  
    
- prefix  
 
        度量标准名称的第一个字符，它将过滤返回给那些以提供的字符串开头的度量标准。它可以与<code>group</code>或<code>type</code>参数结合使用。可以在请求中指定多个前缀；多个前缀应该用逗号分隔。前缀匹配也是区分大小写的。  
    
- property  
   
        允许从任何复合度量中仅请求该度量。多个<code>property</code>参数可以组合起来作为OR请求。例如，要仅从所有度量标准类型和组中获得第99和第999百分位数值，可以添加<code>&amp;property=p99_ms&amp;property=p999_ms</code>到您的请求中。这可以结合<code>group</code>，<code>type</code>以及<code>prefix</code>如果有必要。  
    
- compact  
    
        如果为false，则会返回更详细的响应格式。而不是像这样的回应：  
```
 "metrics": [
    "solr.core.gettingstarted",
    {
      "CORE.aliases": {
        "value": ["gettingstarted"]
      },
      "CORE.coreName": {
        "value": "gettingstarted"
      },
      "CORE.indexDir": {
        "value": "/solr/example/schemaless/solr/gettingstarted/data/index/"
      },
      "CORE.instanceDir": {
        "value": "/solr/example/schemaless/solr/gettingstarted"
      },
      "CORE.refCount": {
        "value": 1
      },
      "CORE.startTime": {
        "value": "2017-03-14T11:43:23.822Z"
      }
    }
  ]
```
        
            答案将如下所示：  
          
        
```
"metrics": [
    "solr.core.gettingstarted",
    {
      "CORE.aliases": [
        "gettingstarted"
      ],
      "CORE.coreName": "gettingstarted",
      "CORE.indexDir": "/solr/example/schemaless/solr/gettingstarted/data/index/",
      "CORE.instanceDir": "/solr/example/schemaless/solr/gettingstarted",
      "CORE.refCount": 1,
      "CORE.startTime": "2017-03-14T11:43:23.822Z"
    }
  ]
```
          
    

与其他请求处理程序一样，Metrics API也可以使用该wt参数来定义输出格式。  

### 例子

在“core”组中只请求“counter”类型的度量，以JSON返回：  
```
http://localhost:8983/solr/admin/metrics?type=counter&amp;group=core
```
仅请求以“INDEX”开头的以“XML”返回的“core”组度量标准：  
```
http://localhost:8983/solr/admin/metrics?wt=xml&amp;prefix=INDEX&amp;group=core
```
