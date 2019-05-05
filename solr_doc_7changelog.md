# Solr7的主要变化 

Solr 7 是 Solr 的一个主要的新版本，它引入了新的功能和其他一些可能影响您现有安装的其他更改。

## 新增功能和功能增强

### 复制模式

在Solr7之前，SolrCloud主从复制模型一直允许任何副本在领导者丢失时成为领导者。这对大多数用户非常有效，可在群集出现问题时提供可靠的故障转移。但是，在大型集群中需要付出代价，因为所有副本必须始终保持同步。

为了提供额外的灵活性，我们增加了两种新的复制方式:TLOG＆PULL。这些新类型提供了复制选项，这些复制只能通过复制领导者的索引段来与领导者同步。TLOG类型具有维护事务日志（其名称的“tlog”）的额外好处，这将允许它在必要时恢复并成为领导者; PULL类型不维护事务日志，因此无法成为领导者。

作为此更改的一部分，传统类型的副本现在名为NRT。如果未明确定义许多TLOG或PULL副本，则Solr默认创建NRT副本。如果此模型适合您，您将不必更改任何内容

### Solr autoscaling

Solr autoscaling是 Solr中的一套新功能，可以更轻松，更自动地管理SolrCloud集群。  
autoscaling是的核心是为用户提供规则语法，以定义如何在集群中分配节点和分片的首选项和策略，目的是在集群中保持平衡。从Solr 7开始，Solr将在确定使用各种Collections API命令创建或移动的新分片和副本的放置位置时考虑任何策略或首选项规则

### 其他功能和增强

- Analytics 组件已被重构。该组件的文档正在进行中；在可用之前，请参阅 SOLR-11144 了解更多详情。
- 在早期的 6.x 版本中发布了其他几个新功能，您可能会错过这些功能：
  - 学习排名（Learning to Rank）。
  - 统一的高亮（Unified Highlighter）。
  - 度量 API（Metrics API）。另请参阅下面的 “JMX 支持” 和 “MBean”一节中有关弃用的信息。
  - 有效负载查询（Payload queries）。
  - 流媒体评估器（Streaming Evaluators）。 
  - / v2 API。
  - 图流表达式（Graph streaming expressions）。


## 配置和默认更改

### 新的默认配置集

对Solr附带的configSets进行了一些更改；不仅他们的内容，还有 Solr 在这些方面的行为：

- data_driven_configset 与 basic_configset已被删除，替代的是_defaultconfigset。sample_techproducts_configset还保留，与example/exampledocs目录中的Solr附带的示例文档一起使用。
- 当创建一个新的集合时，如果未指定一个configSet，_default 将被使用。
  - 如果你使用 SolrCloud，_defaultconfigSet 会自动上传到 ZooKeeper。
  - 如果使用独立模式，将使用_defaultconfigSet作为基础自动创建instanceDir。


### Schemaless的改进

为了改进Schemaless的功能，当检测到传入字段中的数据应该具有基于文本的字段类型时，Solr 现在的行为会有所不同。

- text_general默认情况下，传入的字段将被编入索引（您可以更改此字段）。该字段的名称将与文档中定义的字段名称相同。
- 将在模式中插入复制字段规则，以将新text_general字段复制到具有名称的新字段<name>_str。该字段的类型将是一个strings字段（允许多个值）。文本字段的前256个字符将插入到新strings字段中。

如果要删除复制字段规则，或更改插入字符串字段的字符数或使用的字段类型，可以自定义此行为。有关详细信息，请参阅 “[Schemaless](solr_doc_schemaless.md)”

>Tip：由于复制字段规则可以降低索引的速度并增加索引大小, 所以建议您在需要时只使用复制字段。如果您不需要对字段进行排序或分面，则应该删除自动生成的复制字段规则。

可以使用 update.autoCreateFields 属性禁用自动字段创建。如下,通过命令调用 Config API实现：

```sh
curl http://host:8983/solr/mycollection/config -d '{"set-user-property": {"update.autoCreateFields":"false"}}'
```

### 默认行为的更改

- JSON 现在是默认的响应格式。如果您依赖于 XML 响应，您现在必须在请求中定义：wt=xml。另外，行缩进是默认启用的（indent=on）。
- sow 参数（“Split on Whitespace”的缩写）现在默认为 false，允许支持多字同义词。该参数与 eDismax和标准/“lucene” 查询解析器一起使用。如果此参数没有明确指定为：true，查询文本将以空白字符拆分。
- legacyCloud 参数现在默认为 false。如果副本的项不存在 state.json，则该副本将不会被注册。这可能会影响到复制副本的用户，并自动将其注册为分片的一部分。通过使用以下命令在群集属性中设置属性 legacyCloud=true，可以退回到旧的行为：

```sh
./server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:2181 -cmd clusterprop -name legacyCloud -val true
```

- 如果 solrconfig 中的 luceneMatchVersion 是 7.0.0 或以上版本，则 eDismax 查询分析器参数 lowercaseOperators现在默认为 false。luceneMatchVersion 低于7.0.0仍然是true。这意味着客户端必须以大写的方式发送布尔运算符（如 AND、OR 和 NOT）才能被识别，或者您必须明确地设置该参数为 true。
- 如果 luceneMatchVersion 是7.0.0 或以上，则 solrconfig 中的 handleSelect 参数现在默认为 false。这会导致 Solr 忽略 qt 参数，如果它存在于请求中。如果您的请求处理程序没有前导 '/'，则可以设置 handleSelect="true" 或考虑迁移您的配置。该 qt 参数仍用作指定要使用的请求处理程序（尾部 URL 路径）的 SolrJ特殊参数。
- lucenePlusSort 查询解析器（又名 “旧 Lucene 查询解析器”）已被弃用，不再隐式定义。如果你想继续使用这个解析器直到 Solr 8（将被删除），你必须将它注册到您的 solrconfig. xml 中，如：&lt;queryParser name="lucenePlusSort" class="solr.OldLuceneQParserPlugin"&gt;
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

- solrconfig.xml 中的 &lt;jmx&gt; 元素已被删除， 替代的是solr中定义的 &lt;metrics&gt; &lt;reporter&gt; 元素。如果缺少 SolrJmxReporter，并且在找到本地 MBean 服务器时自动添加默认的实例，则会提供有限的向后兼容性。本地 MBean 服务器可以通过 solr.in.sh 中的 ENABLE_REMOTE_JMX_OPTS 或者经由系统的性能（例如：Dcom.sun.management.jmxremote）被激活。此默认实例将所有注册表中的所有Solr 度量标准导出为分层 MBean。还可以通过指定SolrJmxReporter 配置来禁用该行为，该配置使用的布尔型 init 参数设置为 false。对于更加 fine-grained 的控制，用户应明确指定至少一个 SolrJmxReporter配置。另请参阅 &lt;metrics&gt; &lt;reporter&gt; 元素部分，其中介绍了如何设置 Metrics Reporter solr.xml。请注意，后向兼容性支持可能会在 Solr 8 中删除。
- MBean 名称和属性现在遵循度量中使用的分层名称。这也反映在：/admin/mbeans和/admin/plugins 输出中，并且可以在 UI Plugins 选项卡中观察到，因为现在所有这些 API 都从metrics API 获取其数据。旧的（大部分是扁平的）JMX 视图已被删除。


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
