## Solr模式元素 
<div class="content-intro view-box ">本节介绍在前面几节中未提及的 schema.xml 的其他几个重要的模式元素。  

## 唯一的键<a href="http://lucene.apache.org/solr/guide/7_0/other-schema-elements.html#unique-key"/>

uniqueKey 元素指定哪个字段是文档的唯一标识符。虽然 uniqueKey 不是必需的，但它几乎总是由您的应用程序设计。例如，如果您将在索引中更新文档，则应使用 uniqueKey。  
  
您可以通过命名来定义唯一的关键字段：  
```
&lt;uniqueKey&gt;id&lt;/uniqueKey&gt;
```
架构默认值和 copyFields 不能用于填充 uniqueKey 字段。在 fieldType 中 uniqueKey 不得进行分析。您可以使用 UUIDUpdateProcessorFactory 自动生成具有 uniqueKey的值。  
  
此外，如果 uniqueKey 字段被使用，则该操作将失败，但是是多值的（或者从 fieldtype 继承了多值性）。但是，只要适当地使用该字段，uniqueKey 将继续工作。  

## 相似<a href="http://lucene.apache.org/solr/guide/7_0/other-schema-elements.html#similarity"/>

相似性是一个 Lucene 类，用于在搜索中对文档进行评分。  
  
每个集合都有一个“全局”相似性，默认情况下，Solr 使用一个隐式 SchemaSimilarityFactory，它允许将单个字段类型配置一个“每类型”特定的相似性，并隐式使用 BM25Similarity 对任何字段类型没有明确的相似性。  
可以通过在 schema.xml 中的顶级 &lt;similarity/&gt; 元素(在任何单个字段类型之外) 来重写此默认行为。这个相似性声明可以直接引用具有无参数构造函数的类的名称，如下例中显示 BM25Similarity：  
```
&lt;similarity class="solr.BM25SimilarityFactory"/&gt;
```
或者引用 SimilarityFactory 实现，它可能采用可选的初始化参数:：  
```
&lt;similarity class="solr.DFRSimilarityFactory"&gt;
  &lt;str name="basicModel"&gt;P&lt;/str&gt;
  &lt;str name="afterEffect"&gt;L&lt;/str&gt;
  &lt;str name="normalization"&gt;H2&lt;/str&gt;
  &lt;float name="c"&gt;7&lt;/float&gt;
&lt;/similarity&gt;
```
在大多数情况下， 如果您的 schema.xml 还包括字段类型特定的 &lt;similarity/&gt; 声明，则指定全局级别相似性会导致错误。一个重要的例外是，您可以明确地声明一个  SchemaSimilarityFactory，并指定默认行为对于所有不使用字段类型名称 (由 defaultSimFromFieldType 指定) 声明显式相似性的字段类型。具有特定相似性的配置：  
```
&lt;similarity class="solr.SchemaSimilarityFactory"&gt;
  &lt;str name="defaultSimFromFieldType"&gt;text_dfr&lt;/str&gt;
&lt;/similarity&gt;
&lt;fieldType name="text_dfr" class="solr.TextField"&gt;
  &lt;analyzer ... /&gt;
  &lt;similarity class="solr.DFRSimilarityFactory"&gt;
    &lt;str name="basicModel"&gt;I(F)&lt;/str&gt;
    &lt;str name="afterEffect"&gt;B&lt;/str&gt;
    &lt;str name="normalization"&gt;H3&lt;/str&gt;
    &lt;float name="mu"&gt;900&lt;/float&gt;
  &lt;/similarity&gt;
&lt;/fieldType&gt;
&lt;fieldType name="text_ib" class="solr.TextField"&gt;
  &lt;analyzer ... /&gt;
  &lt;similarity class="solr.IBSimilarityFactory"&gt;
    &lt;str name="distribution"&gt;SPL&lt;/str&gt;
    &lt;str name="lambda"&gt;DF&lt;/str&gt;
    &lt;str name="normalization"&gt;H2&lt;/str&gt;
  &lt;/similarity&gt;
&lt;/fieldType&gt;
&lt;fieldType name="text_other" class="solr.TextField"&gt;
  &lt;analyzer ... /&gt;
&lt;/fieldType&gt;
```
在上面的示例中，IBSimilarityFactory（使用基于信息的模型）将用于 text_ib 类型的任何字段，而 DFRSimilarityFactory（随机的分歧）将用于 text_dfr 类型的任何字段，以及使用类型的任何字段，该类型没有明确指定 &lt;similarity/&gt;。  
  
如果 SchemaSimilarityFactory 是通过配置 defaultSimFromFieldType 显式声明的，则 BM25Similarity 将隐式用作默认值。  
  
除了此页面上提到的各个工厂，还有其他几种类似的实现，如：SweetSpotSimilarityFactory、ClassicSimilarityFactory 等等。有关详细信息，请参见 Solr Javadocs 的相似工厂。  
  
  
