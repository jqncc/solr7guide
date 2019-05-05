## Solr的架构API 
<div class="content-intro view-box ">Solr 架构 API 允许您使用 HTTP API 来管理架构中的许多元素。
      
  
Schema API 使用 ManagedIndexSchemaFactory 类，这是现代 Solr 版本中的默认架构工厂。有关为索引选择架构工厂的更多信息，请参阅 SolrConfig 中的架构工厂定义一节。  
此 API 为每个集合 (或使用独立 solr 时的核心) 提供对 Solr 架构的读取和写入访问权限。支持对所有架构元素的读访问权限。字段、动态字段、字段类型和 copyField规则可以添加、删除或替换。将来的 Solr 版本将扩展写访问权限，以允许修改更多的架构元素。  
为什么不鼓励手动编辑托管架构？在示例配置中名为 "托管架构（managed-schema）" 的文件可能包括一个建议不要手动编辑文件的注释。在架构 API 存在之前, 此类编辑是对架构进行更改的唯一方法，用户可能会强烈的希望继续以这种方式进行更改。  
之所以不鼓励这样做，是因为如果后面描述的架构 API 被用来进行更改，则可能会丢失对架构的手工编辑，除非在使用架构 API 之前重新加载了核心或启动了Solr。如果在手动编辑后总是重新加载或重新启动, 那么执行这些编辑就没有问题了。  
API 允许对所有调用的两种输出模式：JSON或XML。在请求完整的架构时，还有另一个输出模式，它是以 xml 格式在托管架构文件本身之后建模的。  
在使用 API 修改架构时，将自动进行核心重新加载，以便随后对其索引的文档立即进行更改。以前编入索引的文档不会自动更新 - 如果现有索引数据使用您更改的架构元素，则必须索引它们。
      
  
修改架构后重新索引！如果修改架构，您可能需要重新索引所有文档。如果不这样做，您可能会失去对文档的访问权限，或者无法正确解释它们，例如在替换字段类型后。
      
修改您的架构将永远不会修改任何已经索引的文档。您必须重新索引文档以便将架构更改应用于这些文档。更改后所做的查询和更新可能会遇到在更改之前不存在的错误。完全删除索引并重建它通常是解决此类错误的唯一选项。
      
  
API 的基本地址为 http://&lt;host&gt;: &lt;port&gt;/solr/&lt;collection_name&gt;。例如，如果您运行 Solr 的 "cloud" 示例 (通过下面显示的 bin/solr 命令), 创建 "gettingstarted" 集合, 则该集合的基本 URL（如本节中的所有示例URL）将是：http://localhost:8983/solr/gettingstarted。  
```
bin/solr -e cloud -noprompt
```

## 架构 API 入口点<a href="http://lucene.apache.org/solr/guide/7_0/schema-api.html#schema-api-entry-points"/>
  
- /schema：检索架构，或修改架构以添加、删除或替换字段、动态字段、复制字段或字段类型。
- /schema/fields：检索有关所有定义的字段或特定命名字段的信息。
- /schema/dynamicfields：检索有关所有动态字段规则或特定命名动态规则的信息。
- /schema/fieldtypes：检索有关所有字段类型或特定字段类型的信息。
- /schema/copyfields：检索有关复制字段的信息。
- /schema/name：检索模式名称。
- /schema/version：检索模式版本。
- /schema/uniquekey：检索已定义的 uniqueKey。
- /schema/similarity：检索全局相似性定义。
