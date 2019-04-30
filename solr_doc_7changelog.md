# Solr7的主要变化 

Solr 7 是 Solr 的一个主要的新版本，它引入了新的功能和其他一些可能影响您现有安装的其他更改。

## 开始升级

在开始迁移您的配置和索引之前，需要考虑 Solr 7 中的主要更改。此页面旨在突出显示最大的变化 - 您可能需要了解的新功能，还包括默认行为和已删除的已否决功能的更改。  
然而，Solr 7 中有许多变化，因此，对 Solr 升级说明以及 Solr 实例中的 [CHANGES.txt ](https://lucene.apache.org/solr/7_0_0//changes/Changes.html)文件进行彻底的审查将有助于您计划向 Solr 7 迁移。本节将重点介绍 Solr 7中的一些你应该知道的重大变化。  
您还应该考虑在尚未升级到的任何版本中对 Solr 所做的所有更改。例如，如果您当前正在使用 Solr 6.2，则除了 7.0 的更改之外，还应该查看所有后续 6.x 版本中所做的更改。  
将数据重新编入索引被认为是最佳做法，如果可能的话，您应该尝试这样做。但是，如果重新索引不可行，请记住，您只能一次升级一个主要版本。因此，Solr 6.x 索引将与 Solr 7 兼容，但 Solr 5.x 索引不会。  
如果您现在不重新编制索引，请记住，您将需要重新索引数据或升级索引，然后才能在将来发布 Solr 8 时转移到 Solr 8。有关如何升级索引的更多详细信息，请参阅IndexUpgrader 工具一节。  
有关如何升级 SolrCloud 群集的详细信息，另请参阅升级 Solr 群集一节。  

## 新增功能和功能增强


### 复制模式

直到 Solr 7 ，SolrCloud 模型的复制副本允许任何复制副本在领导者丢失时成为领导者。这对大多数用户来说非常有效，在集群出现问题的情况下提供可靠的故障切换。但是，由于所有副本必须在任何时候都同步，因此在大型群集中需要付出代价。  
为了提供更多的灵活性，已经添加了两种新类型的副本，名为 TLOG＆PULL。这些新类型提供了一些选项，以便仅通过从前导项复制索引段与引线同步的副本。TLOG 类型还有一个额外的好处，就是维护一个事务日志（它的名字为 “tlog”），如果需要的话，它可以恢复并成为领导者；PULL 类型不维护事务日志，因此不能成为领导者。  
作为这种改变的一部分，传统的副本现在被命名为 NRT。如果您没有明确定义一些 TLOG 或 PULL 副本，则 Solr 默认创建 NRT 副本。如果这个模型适合您的工作，您不需要改变任何东西。  
有关新副本模式的更多详细信息，以及如何在群集中定义副本类型，请参见副本类型一节。  

### 自动缩放

Solr 自动缩放是 Solr 中的一个新功能套件，用于管理 SolrCloud 群集更加简单和自动化。  
Solr 自动缩放的核心是为用户提供一个规则语法来定义如何在集群中分发节点和碎片的首选项和策略，目的是在集群中保持平衡。从 Solr 7 开始，Solr 将在确定将创建或移动各种 Collections API 命令的新碎片和副本放置到何处时，将考虑任何策略或首选项规则。  
有关 Solr 7.0 中可用选项的详细信息，请参见 SolrCloud 自动缩放部分。预计在该领域的后续 7.x 版本中将会发布更多功能。  

### 其他功能和增强


    - Analytics 组件已被重构。该组件的文档正在进行中；在可用之前，请参阅 SOLR-11144 了解更多详情。
    - 在早期的 6.x 版本中发布了其他几个新功能，您可能会错过这些功能：学习排名（Learning to Rank）。统一的高亮（Unified Highlighter）。度量 API（Metrics API）。另请参阅下面的 “JMX 支持” 和 “MBean”一节中有关弃用的信息。有效负载查询（Payload queries）。流媒体评估器（Streaming Evaluators）。 / v2 API。 图流表达式（Graph streaming expressions）。


## 配置和默认更改


### 新的默认配置集

对与 Solr 有关的配置进行了若干修改；不仅他们的内容，还有 Solr 在这些方面的行为：
      
  

    - data_driven_configset 与 basic_configset 已被删除，取而代之的是 _defaultconfigset。sample_techproducts_configset 还保留，专门与example/exampledocs 目录中的 Solr 附带的示例文档一起使用。
    - 当创建一个新的集合时，如果您不指定一个 configSet，_default 将被使用。如果你使用 SolrCloud，_defaultconfigSet 会自动上传到 ZooKeeper。如果使用独立模式，instanceDir 将自动创建，使用 _defaultconfigSet 作为基础。


### 无模式的改进

为了改进无模式的功能，当检测到传入字段中的数据应该具有基于文本的字段类型时，Solr 现在的行为会有所不同。
      
  

    - 默认情况下，传入的字段将被索引为 text_general（可以更改）。该字段的名称将与文档中定义的字段名称相同。
    - 复制字段规则将被插入到您的模式中，以将新的 text_general 字段复制到具有名称的新字段 &lt;name&gt;_str。这个字段的类型将是一个 strings 字段（允许多个值）。文本字段的前 256 个字符将被插入到新的字符串字段中。

如果您希望删除复制字段规则，或者更改插入到字符串字段的字符数或所使用的字段类型，则可以自定义此行为。有关详细信息，请参阅 “无模式”部分。  
```
Tip：由于复制字段规则可以降低索引的速度并增加索引大小, 所以建议您在需要时只使用复制字段。如果您不需要对字段进行排序或分面，则应该删除自动生成的复制字段规则。
```

可以使用 update.autoCreateFields 属性禁用自动字段创建。要做到这一点，您可以通过如下命令使用 Config API 例如：  
```
curl http://host:8983/solr/mycollection/config -d '{"set-user-property": {"update.autoCreateFields":"false"}}'
```


### 对默认行为的更改


    - JSON 现在是默认的响应格式。如果您依赖于 XML 响应，您现在必须在请求中定义：wt=xml。另外，行缩进是默认启用的（indent=on）。
    - sow 参数（“Split on Whitespace”的缩写）现在默认为 false，允许支持多字同义词。该参数与 eDismax 和 standard/“lucene” 查询解析器一起使用。如果此参数没有明确指定为：true，查询文本将不会在空白上拆分。
    - legacyCloud 参数现在默认为 false。如果副本的项不存在 state.json，则该副本将不会被注册。这可能会影响到复制副本的用户，并自动将其注册为分片的一部分。通过使用以下命令在群集属性中设置属性 legacyCloud=true，可以退回到旧的行为：
```
./server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:2181 -cmd clusterprop -name legacyCloud -val true
```

    
    - 如果 solrconfig 中的 luceneMatchVersion 是 7.0.0 或以上版本，则 eDismax 查询分析器参数 lowercaseOperators 现在默认为 false。luceneMatchVersion 低于7.0.0 的行为是不变的（因此，为 true）。这意味着客户端必须以大写的方式发送布尔运算符（如 AND、OR 和 NOT）才能被识别，或者您必须明确地设置该参数为 true。
    - 如果 luceneMatchVersion 是7.0.0 或以上，则 solrconfig 中的 handleSelect 参数现在默认为 false。这会导致 Solr 忽略 qt 参数，如果它存在于请求中。如果您的请求处理程序没有前导 '/'，则可以设置 handleSelect="true" 或考虑迁移您的配置。该 qt 参数仍用作指定要使用的请求处理程序（尾部 URL 路径）的 SolrJ特殊参数。
    - lucenePlusSort 查询解析器（又名 “旧 Lucene 查询解析器”）已被弃用，不再隐式定义。如果你想继续使用这个解析器直到 Solr 8（当它将被删除），你必须将它注册到您的 solrconfig. xml 中，如：
```
&lt;queryParser name="lucenePlusSort" class="solr.OldLuceneQParserPlugin"/&gt;
```

    
    - TemplateUpdateRequestProcessorFactory 名称从 Template更改为 template，AtomicUpdateProcessorFactory 的名称从 Atomic 改为atomic，此外，TemplateUpdateRequestProcessorFactory 现在使用 {} 而不是 $ {} 作为模板。


## 弃用和删除的功能


### 点字段是默认的数值类型

Solr 全面实现了 * PointField 类型，取代了基于 Trie * 的数字字段。现在所有的 Trie * 字段都被认为是不赞成使用的，并将在 Solr 8 中删除。  
如果您在您的模式使用 Trie * 字段，则您应该考虑尽快迁移到 PointFields。更改为新的 PointField 类型将要求您重新索引数据。  

### 空间领域

以下与空间相关的字段已被弃用：  

    - LatLonType
    - GeoHashField
    - SpatialVectorFieldType
    - SpatialTermQueryPrefixTreeFieldType

改为选择下列字段类型之一：  

    - LatLonPointSpatialField
    - SpatialRecursivePrefixTreeField
    - RptWithGeometrySpatialField

有关更多信息，请参阅空间搜索部分。  

### JMX 支持和 MBeans


    - solrconfig.xml 中的 &lt;jmx&gt; 元素已被删除， 以支持在 solr 中定义的 &lt;metrics&gt; &lt;reporter&gt; 元素。如果缺少 SolrJmxReporter，并且在找到本地 MBean 服务器时自动添加默认的实例，则会提供有限的向后兼容性。本地 MBean 服务器可以通过 solr.in.sh 中的 ENABLE_REMOTE_JMX_OPTS 或者经由系统的性能（例如：Dcom.sun.management.jmxremote）被激活。此默认实例将所有注册表中的所有
        Solr 度量标准导出为分层 MBean。还可以通过指定SolrJmxReporter 配置来禁用该行为，该配置使用的布尔型 init 参数设置为 false。对于更加 fine-grained 的控制，用户应明确指定至少一个 SolrJmxReporter配置。另请参阅 &lt;metrics&gt; &lt;reporter&gt; 元素部分，其中介绍了如何设置 Metrics Reporter solr.xml。请注意，后向兼容性支持可能会在 Solr 8 中删除。
          
    
    - MBean 名称和属性现在遵循度量中使用的分层名称。这也反映在：/admin/mbeans和/admin/plugins 输出中，并且可以在 UI Plugins 选项卡中观察到，因为现在所有这些 API 都从度量 API 获取其数据。旧的（大部分是平坦的）JMX 视图已被删除。


### SolrJ

SolrJ 进行了以下更改。  

    - HttpClientInterceptorPlugin：现在是 HttpClientBuilderPlugin，必须与一个 SolrHttpClientBuilder 一起工作，而不是一个 HttpClientConfigurer。
    - HttpClientUtil：现在允许通过 SolrHttpClientBuilder 配置 HttpClient 实例，而不是通过一个 HttpClientConfigurer。使用 env 变量SOLR_AUTHENTICATION_CLIENT_CONFIGURER 不再有效，请使用 SOLR_AUTHENTICATION_CLIENT_BUILDER。
    - SolrClient：实现现在使用自己的内部配置套接字超时、连接超时并允许重定向，而不是在生成 HttpClient 实例时设置为默认值。在 SolrClient 实例上使用适当的设置器。
    - HttpSolrClient#setAllowCompression 已被移除，必须将压缩作为构造函数参数启用。
    - HttpSolrClient#setDefaultMaxConnectionsPerHost 和 HttpSolrClient#setMaxTotalConnections 已被删除。现在这些默认值非常高，只能在创建HttpClient 实例时通过参数进行更改。


### 其他的弃用和删除


    - 模式中的 defaultOperator 参数不再被支持，改用 q.op 参数。这个选项已经被弃用了几个版本。有关更多信息，请参见标准查询解析器参数部分。
    - 模式中的 defaultSearchField 参数不再被支持，改用 df 参数。这个选项已经被弃用了几个版本。有关更多信息，请参见标准查询解析器参数部分。
    - mergePolicy、mergeFactor 和 maxMergeDocs 参数已被删除并不再支持。你应该定义一个 mergePolicyFactory。有关更多信息，请参见mergePolicyFactory 部分。
    - PostingsSolrHighlighter 已被弃用。建议您改为使用 UnifiedHighlighter。有关更多信息，请参阅 Unified Highlighter 部分。
    - 索引时间提升已经从 Lucene 中删除，并且不再可以从 Solr 获得。如果提供了任何提升，它们将被索引链忽略。作为替代，索引时间评分因子应该在单独的字段中编入索引，并使用函数查询与查询评分相结合。有关更多信息，请参阅函数查询一节。
    - StandardRequestHandler 已被弃用，改为使用 SearchHandler。
    - 为了提高集合 API 参数的一致性，MOVEREPLICA 命令和源的参数名称为 fromNode，REPLACENODE 命令中的 target 已被弃用，取而代之的是 sourceNode和 targetNode。旧名称将继续为后向兼容工作，但他们将在 Solr 8 中被删除。
    - 未使用的 valType 选项已从 ExternalFileField 中删除，如果在模式中有这个选项，则可以安全地删除它。


## 早期的 6.x 版本的主要变化

以下对早期 Solr 6.x 版本中的更改的摘要突出显示了在本指南的早期版本中列出的 Solr 6.0 和 6.6 之间发布的重大更改。如上述各节所述，在 Solr 7 中提到弃用可能会被取消。
      
  
请再次注意，这并不是可能影响您的安装的所有更改的完整列表，因此如果从早于 6.6 的任何版本升级，强烈建议您查看 CHANGES.txt。  

    - Solr 的贡献：map-reduce、morphlines-core 和 morphlines-cell 已被删除。
    - JSON Facet API 现在使用超级日志记录进行 numBuckets 基数计算，并在将存储区过滤之前计算 mincount 大于1的基数。
    - 如果您使用历史日期，特别是 1582 年或之前的日期，则应该重新编制索引，以便更好地处理日期。
    - 如果您使用 method=stream 的 JSON Facet API（json.facet），您现在必须设置 sort='index asc' 以获取流式传输行为；否则不会流出。提醒：method 是一个提示，不会改变其他参数的默认值。
    - 如果您使用 JSON Facet API（json.facet）来面向数字字段，并且如果您使用 mincount=0 或者如果您设置了前缀，那么现在将出现错误，因为这些选项与数字分面不兼容。
    - Solr 在 INFO 级别的日志记录详细程度已大大降低，您可能需要更新日志配置以使用 DEBUG 级别来查看以前在 INFO 级别上查看的所有日志记录消息。
    - 我们不再支持 solr.log 和 solr_gc.log 文件在日期戳的副本。如果您依赖于 "日志" 文件夹中的 solr_log_&lt;date&gt; 或 solr_gc_log_&lt;date&gt;，将不再是这种情况。有关如何在 Solr 6.3 中进行日志轮换的详细信息，请参阅配置日志记录一节。
    - MiniSolrCloudCluster 中的 create / deleteCollection 方法已被弃用。用户应该使用 CollectionAdminRequest API。此外，MiniSolrCloudCluster#uploadConfigDir(File, String)已被弃用，以支持 #uploadConfigSet(Path, String)。
    - 现在，默认情况下，bin/solr.in.sh（Windows 上为 bin/solr.in.cmd）已完全注释。以前，情况并非如此，它具有掩盖现有环境变量的作用。
    - _version_ 字段不再被编入索引，现在 indexed=false 默认定义，因为该字段已启用 DocValues。
    - /export 处理程序已被更改，因此它不再返回原始文档中不存在的数值字段的零 (0)。这种改变的一个后果是，你必须知道，如果原始文档中没有任何元组，那么一些元组将没有值。
    - org.apache.solr.util.stats 中与度量相关的类已被删除，转而支持 Dropwizard 度量库。任何使用这些类的自定义插件都应该更改为使用度量库中的等效类。作为其中的一部分，对监督状态 API 的输出进行了以下更改：
        <ul style="list-style-type:circle">
            <li>“totalTime” 指标已被删除，因为它不再受支持。
            - 监督状态 API 中的度量标准：“75thPctlRequestTime”、“95thPctlRequestTime”、“99thPctlRequestTime” 和 “999thPctlRequestTime” 已被重命名为：“75thPcRequestTime”、“95thPcRequestTime” 等，以与 Solr 的其他部分输出的统计信息保持一致。
            - “avgRequestsPerMinute”、“5minRateRequestsPerMinute” 和 “15minRateRequestsPerMinute” 的度量标准被相应的每秒速率 viz 替代，“avgRequestsPerSecond”、“5minRateRequestsPerSecond” 和 “15minRateRequestsPerSecond” 与 Solr其他部分输出的统计信息保持一致。
        
    </li>
    <li>新增了 UnifiedHighlighter。建议您通过设置 hl.method=unified 和报告反馈来尝试 UnifiedHighlighter。这将很有效率并且更快。hl.useFastVectorHighlighter现在被认为是代替 hl.method=fastVector。</li>
    <li>maxWarmingSearchers 参数现在默认为1，而更重要的是，如果超出此限制而不是引发异常 ，现在就会阻止它的提交。因此，在重叠提交中不再存在风险。尽管如此，用户应该继续避免过多的提交。建议用户从其 solrconfig. xml 文件中删除任何预先存在的 maxWarmingSearchers 条目。</li>
    <li>复杂的短语查询分析器现在支持领先的通配符。注意其可能的沉重程度，鼓励用户在索引时间分析中使用 ReversedWildcardFilter。</li>
    <li>JMX 度量标准 “avgTimePerRequest”（以及每个处理程序的度量 API 中的相应度量）过去是基于总累计时间和请求数的简单 non-decaying 平均值。Codahale度量标准的实现对这个值应用了指数衰减，这个值在最后5分钟内严重偏离了平均值。</li>
    <li>并行 SQL 现在使用 Apache Calcite 作为其 SQL 框架。作为此更改的一部分，默认聚合模式已更改为 facet 而不是 map_reduce。对 SQL 聚合响应和一些 SQL语法更改也进行了更改。有关完整的细节，请参阅并行 SQL 接口文档。</li></ul>
