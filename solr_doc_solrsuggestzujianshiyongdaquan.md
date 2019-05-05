## Solr：suggest组件使用大全 
<div class="content-intro view-box ">
## Suggester

Solr 中的 SuggestComponent 为用户提供查询 term 的自动建议。
      
  
您可以使用此功能在搜索应用程序中实现强大的自动建议功能。  
尽管可以使用[拼写检查](https://www.w3cschool.cn/solr_doc/solr_doc-j9lk2gxe.html)功能为自动建议行为提供支持，但 Solr 为此功能专门设计了一个 SuggestComponent。  
这种方法利用了 Lucene 的 Suggester 实现，并支持 lucene 中所有可用的查找实现。  
这个 Suggester 的主要特点是：  

    - 查找实现可插入性
    - term 字典可插入性，使您可以灵活地选择词典实现
    - 分布式支持

在 Solr 的 "techproducts" 示例中找到的 solrconfig.xml 已经配置了一个 Suggester 实现。有关搜索组件的更多信息，请参阅 SolrConfig 中的 RequestHandlers 和SearchComponents 部分。  
该 techproducts 示例 solrconfig.xml 有一个 suggest 搜索组件和一个已经配置的 /suggest 请求处理程序。您可以将其用作您的配置的基础，或者从头开始创建它，如下所述。  

## 添加建议搜索组件<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#adding-the-suggest-search-component"/>

第一步是将搜索组件添加到 solrconfig. xml，并告诉它使用 SuggestComponent。下面是一些可以使用的示例代码：  
```
&lt;searchComponent name="suggest" class="solr.SuggestComponent"&gt;
  &lt;lst name="suggester"&gt;
    &lt;str name="name"&gt;mySuggester&lt;/str&gt;
    &lt;str name="lookupImpl"&gt;FuzzyLookupFactory&lt;/str&gt;
    &lt;str name="dictionaryImpl"&gt;DocumentDictionaryFactory&lt;/str&gt;
    &lt;str name="field"&gt;cat&lt;/str&gt;
    &lt;str name="weightField"&gt;price&lt;/str&gt;
    &lt;str name="suggestAnalyzerFieldType"&gt;string&lt;/str&gt;
    &lt;str name="buildOnStartup"&gt;false&lt;/str&gt;
  &lt;/lst&gt;
&lt;/searchComponent&gt;
```

### 建议搜索组件参数<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#suggester-search-component-parameters"/>

建议搜索组件采用多个配置参数。
      
  
查找实现的选择（lookupImpl，在建议字典中如何找到 term）和字典实现（dictionaryImpl，term 如何存储在建议字典中）将决定所需的一些参数。  
下面是无论使用什么查找或词典实现，都可以使用的主要参数。在下面的部分中提供了每个实现的附加参数。  
- searchComponent name 参数  
    
        搜索组件的任意名称。  
    
- name 参数  
  
        这个建议的象征性名称。您可以在 URL 参数和 SearchHandler 配置中引用此名称。一个<code>solrconfig.xml</code>文件中可能有多个这样的字符。  
    
- lookupImpl 参数  
   
        查找实现。有几种可能的实现，在下面的查找实现一节中描述。如果没有设置，默认查找是<code>JaspellLookupFactory</code>。  
    
- dictionaryImpl 参数  
   
        要使用的字典实现。有几种可能的实现，在“ 字典实现 ”一节中有描述。  
        
            如果没有设置，默认的字典实现是<code>HighFrequencyDictionaryFactory</code>。但是，如果使用 <code>sourceLocation</code>，字典的实现将会是<code>FileDictionaryFactory</code>。  
          
    
- field 参数  
  
        从索引中使用的字段作为建议 term 的基础。如果<code>sourceLocation</code>为空（表示除了 FileDictionaryFactory 以外的任何字典实现），则将使用来自索引中的该字段的 term。
              
          
   
        
            作为建议的依据，必须存储该字段。您可能希望使用 copyField 规则来创建由文档中其他字段的 term 组成的特殊“建议”字段。无论如何，您很可能只需要对该字段进行最少量的分析，因此另外一个选择是在架构中创建一个只使用基本标记或过滤器的字段类型。这里显示了这种字段类型的一个选项：  
```
&lt;fieldType class="solr.TextField" name="textSuggest" positionIncrementGap="100"&gt;
  &lt;analyzer&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.StandardFilterFactory"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
          
        
            但是，如果您想要根据 term 进行更多的分析，则不需要进行这种最低限度的分析。但是，如果使用<code>AnalyzingLookupFactory</code>作为 <code>lookupImpl</code>，则可以选择定义用于索引和查询时间分析的字段类型规则。  
          
    
- sourceLocation 参数  
   
        字典文件的路径，如果使用<code>FileDictionaryFactory</code>。如果这个值是空的，那么主索引将被用作 term 和权重的来源。  
    
- storeDir 参数  

        存储字典文件的位置。  
    
- buildOnCommit 和 buildOnOptimize 参数  
    
        如果<code>true</code>，则将在 soft-commit 后重建查找数据结构。如果为<code>false</code>默认情况下，查询数据将仅在 URL 参数<code>suggest.build=true</code>请求时才被构建。使用<code>buildOnCommit</code>可以用每个 soft-commit 来重建字典，或<code>buildOnOptimize</code>仅在索引被优化时才建立字典。  
  
        
            一些查找实现可能需要很长时间才能构建，特别是对于大型索引。在这种情况下，不建议使用<code>buildOnCommit</code>或<code>buildOnOptimize</code>特别是使用高频率的 softCommits 。建议使用<code>suggest.build=true</code>手动发出请求，以较低的频率构建建议。  
          
    
- buildOnStartup 参数  
   
        如果<code>true,</code>那么当 Solr 启动或内核被重新加载时，查找数据结构将被建立。如果未指定此参数，则建议程序将检查查找数据结构是否存在于磁盘上，如果未找到，则进行构建。  
        
            如果将其设置为<code>true</code>可能会导致核心花费时间较长（或重新加载），因为建议数据结构需要建立，有时可能需要很长时间。通常首选将此设置设置为<code>false</code>默认设置，而使用<code>suggest.build=true</code>发出请求来手动构建建议。  
          
    

### 查找实现（Lookup Implementations）

该 lookupImpl 参数定义用于在建议索引中查找 term 的算法。有几种可能的实现可供选择，有些需要额外的参数进行配置：  

#### AnalyzingLookupFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#analyzinglookupfactory"/>

首先分析传入的文本并将分析后的表格添加到加权的 FST，然后在查找时执行相同的操作。
      
  
该实现使用以下附加属性：  
- suggestAnalyzerFieldType 属性  
    
        用于查询时间和构建时期建议分析的字段类型。  
    
- exactMatchFirst 属性  
    
        如果为<code>true</code>首先返回默认的确切建议，即使它们是前缀或 FST 中有较大权重的其他字符串。  
    
- preserveSep 属性  
    
        如果<code>true</code>（默认），那么令牌之间的分隔符被保留。这意味着建议对标记化非常敏感（例如，baseball 与 base ball 不同）。  
    
- preservePositionIncrements 属性  
    
        如果<code>true</code>，建议将保留位置增量。这意味着，在构建建议时，会保留标记过滤器（例如，当 StopFilter 匹配停用词时）留下空隙的位置。默认是<code>false</code>。  
    

#### FuzzyLookupFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#fuzzylookupfactory"/>

这是一个 suggester，它是 AnalyzingSuggester 的延伸，但本质上是模糊的。相似性由 Levenshtein 算法测量。
      
  
它的实现使用以下附加属性：  
- exactMatchFirst 属性  
    
        如果<code>true</code>（默认），则首先返回确切建议，即使它们是前缀或 FST 中权重较大的其他字符串。  
    
- preserveSep 属性  
    
        如果<code>true</code>（默认），那么令牌之间的分隔符被保留。这意味着建议对标记化非常敏感（例如，baseball 与 base ball 不同）。  
    
- maxSurfaceFormsPerAnalyzedForm 属性  
    
        为单个分析表单保留的最大表面形式数量。当表面形式太多时，我们丢弃最低的权重。  
    
- maxGraphExpansions 属性  
    
        在构建 FST（“索引时间”）时，我们将每个通过令牌流图的路径作为单独的条目添加。这就为一个建议添加了多少扩展的上限。默认是<code>-1</code>这意味着没有限制。  
    
- preservePositionIncrements 属性  
   
        如果<code>true</code>，建议将保留位置增量。这意味着，在构建建议时，会保留标记过滤器（例如，当 StopFilter 匹配停用词时）留下空隙的位置。默认是<code>false</code>。  
    
- maxEdits 属性  
   
        允许的字符串编辑的最大数量。系统的硬性限制是<code>2</code>。默认是<code>1</code>。  
    
- transpositions 属性  
    
        如果<code>true</code>默认，则默认的对换应视为原始编辑操作。  
    
- nonFuzzyPrefix 属性  
   
        必须匹配建议的通用非模糊前缀匹配的长度。默认是<code>1</code>。  
    
- minFuzzyLength 属性  
    
        在任何字符串编辑之前，查询的最小长度将被允许。默认是<code>3</code>。  
    
- unicodeAware 属性  
  
        如果<code>true</code>，则<code>maxEdits</code>、<code>minFuzzyLength</code>、<code>transpositions</code>和<code>nonFuzzyPrefix</code>参数将在 Unicode 代码点（实际字母），而不是字节来测量。默认是<code>false</code>。  
    

#### AnalyzingInfixLookupFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#analyzinginfixlookupfactory"/>

分析输入文本，然后根据索引文本中的任何标记的前缀匹配建议匹配。这为它的字典使用 Lucene 索引。
      
  
它的实现使用以下附加属性。  
- indexPath 属性  
    
        使用<code>AnalyzingInfixSuggester</code>时，您可以提供自己的路径，索引将在其中生成。默认值是<code>analyzingInfixSuggesterIndexDir</code>，并将在您的集合的<code>data/</code>目录中创建。  
    
- minPrefixChars 属性  
    
        使用 PrefixQuery 之前的最小字符数（默认值是<code>4</code>）。比这更短的前缀被索引为字符 ngrams（增加索引大小，但查找速度更快）。  
    
- allTermsRequired 属性  
    
        多个 term 的布尔选项。默认是<code>true</code>，所有的 term 将是必需的。  
    
- highlight 属性  
    
        突出显示 term。默认是<code>true</code>。  
    

这个实现支持上下文过滤。  

#### BlendedInfixLookupFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#blendedinfixlookupfactory"/>

AnalyzingInfixSuggester 的扩展，它为匹配的文档中的权重前缀匹配提供附加功能。您可以将其分数提高，如果命中是接近开始的建议，反之亦然。
      
  
该实现使用以下附加属性：  
- blenderType 属性  
   
        用于使用第一个匹配词的位置计算权重系数。可以是以下之一：  
        
            <ul><li>position_linear  
                
                    weightFieldValue*(1 - 0.10*position)：匹配开始将获得更高的分数。这是默认的。  
                
- position_reciprocal  
                
                    weightFieldValue/(1+position)：匹配结束将获得更高的分数。  
                
                    exponent：position_reciprocal blenderType 的可选配置变量，用于控制分数增加或减少的速度。默认<code>2.0</code>。  
                
            
          
    </li><li>numFactor 属性  
   
        要将结果修剪的搜索元素数乘以的系数。默认值为 10。
              
          
    </li><li>indexPath 属性  
   
        使用<code>BlendedInfixSuggester</code>时，您可以提供自己的路径，索引将在其中生成。默认的目录名称是<code>blendedInfixSuggesterIndexDir</code>，将在您的集合数据目录中创建。  
    </li><li>minPrefixChars 属性  
   
        使用 PrefixQuery 之前的最小字符数（默认值是<code>4</code>）。比这更短的前缀被索引为字符 ngrams（增加索引大小，但查找速度更快）。  
    </li>
</ul>
这个实现支持[上下文过滤](http://lucene.apache.org/solr/guide/7_0/suggester.html#context-filtering)。  

#### FreeTextLookupFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#freetextlookupfactory"/>

它会查看最后一个标记以及用户输入的任何最终标记的前缀（如果存在），以预测最有可能的下一个标记。还可以指定需要考虑的以前标记的数目。当主要建议未能找到任何建议时，此suggester 只能用作后备。
      
  
该实现使用以下附加属性：  
- suggestFreeTextAnalyzerFieldType   
 
        分析器用于“查询时间”和“建立时间”来分析建议。该参数是必需的。  
    
- ngrams  
   
        最大数量的标记数，其中超出的单个将被作为字典。默认值是<code>2</code>。如果增加这个数字，意味着在提出建议时，要比以前的两个标记考虑的要多。  
    

#### FSTLookupFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#fstlookupfactory"/>

基于自动机的查找。此实现的生成速度较慢，但是提供了最低的内存成本。除非您需要更复杂的匹配结果，否则我们推荐使用此实现，在这种情况下，您应该使用 Jaspell 实现。
      
  
该实现使用以下附加属性：  
- exactMatchFirst  
    
        如果<code>true</code>（默认的）首先返回确切建议，即使它们是前缀或 FST 中权重较大的其他字符串。  
    
- weightBuckets  
   
        在建立词典时，建议将使用的权重的单独的 bucket 的数量。  
    

#### TSTLookupFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#tstlookupfactory"/>

一种简单的基于三元线索的查找。  

#### WFSTLookupFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#wfstlookupfactory"/>

一个加权自动机表示，这是一个替代 FSTLookup 的更细致的排序。WFSTLookup不使用 bucket，而是使用最短路径算法。
      
  
请注意，它期望权重是整数。如果重量不足，则认为是1.0。权重在 spellcheck.onlyMorePopular=true 时影响匹配建议的排序：权重被视为“受欢迎程度”分数，较高的权重优先于权重较低的建议。  

#### JaspellLookupFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#jaspelllookupfactory"/>

基于 JaSpell 项目的三元树查找更复杂的查找。如果您需要更复杂的匹配结果，请使用此实现。  

### 字典实现

字典实现定义 term 的存储方式。有几个选项，如果需要，可以在单个请求中使用多个字典。
      
  

#### DocumentDictionaryFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#documentdictionaryfactory"/>

包含 term、权重和从索引中提取的可选有效负载的字典。
      
  
除了通常为 Suggester 描述的参数以及查找实现之外，该字典实现还采用以下参数：  
- weightField  
    
        存储的字段或数字 DocValue 字段。该参数是可选的。  
    
- payloadField  
    
        该<code>payloadField</code>应存储的字段。该参数是可选的。  
    
- contextField  
    
        用于上下文过滤的字段。请注意，只有一些查找实现支持过滤。  
    

#### DocumentExpressionDictionaryFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#documentexpressiondictionaryfactory"/>

此字典实现与 DocumentDictionaryFactory 相同，但允许用户将任意表达式指定到 weightExpression 标记中。
      
除了通常为 Suggester 描述的参数和查找实现之外，此字典实现还采用以下参数:  
- payloadField  
    
        该<code>payloadField</code>应存储的字段。该参数是可选的。  
    
- weightExpression  

        用于对建议进行评分的任意表达式。使用的字段必须是数字字段。此参数是必需的。  
    
- contextField  
   
        用于上下文过滤的字段。请注意，只有一些查找实现支持过滤。  
    

#### HighFrequencyDictionaryFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#highfrequencydictionaryfactory"/>

这个字典的实现允许在非常常见的 term 可能压倒其他 term 的情况下，添加阈值以减少不常用的 term。
      
  
除了通常为 Suggester 描述的参数和查找实现之外，此字典实现还需要一个参数：  
- threshold  
    
        介于0和1之间的值，表示为了添加到查阅字典而应出现的文档总数的最小部分。  
    

#### FileDictionaryFactory<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#filedictionaryfactory"/>

这个字典实现允许使用包含建议条目的外部文件。权重和有效载荷也可以使用。
      
  
如果使用字典文件，它应该是 UTF-8 编码的纯文本文件。您可以在词典文件中同时使用单个 term 和短语。如果添加权重或有效负载，则应使用与 fieldDelimiter 属性一起定义的分隔符（默认值为“\ t”，即制表符）将其与 term 分开。如果使用有效载荷，则文件中的第一行必须指定有效载荷。  
除了通常为 Suggester 描述的参数和查找实现之外，此字典实现还需要一个参数：  
- fieldDelimiter  
    
        指定分隔条目、权重和有效载荷的分隔符。默认是tab（<code>\t</code>）。  
        
            示例文件：  
```
acquire
accidentally    2.0
accommodate 3.0
```
          
    

### 多个词典<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#multiple-dictionaries"/>

可以在单个 SuggestComponent 定义中包含多个 dictionaryImpl 定义。
      
为此，只需定义单独的 suggesters，如以下示例所示:  
```
&lt;searchComponent name="suggest" class="solr.SuggestComponent"&gt;
  &lt;lst name="suggester"&gt;
    &lt;str name="name"&gt;mySuggester&lt;/str&gt;
    &lt;str name="lookupImpl"&gt;FuzzyLookupFactory&lt;/str&gt;
    &lt;str name="dictionaryImpl"&gt;DocumentDictionaryFactory&lt;/str&gt;
    &lt;str name="field"&gt;cat&lt;/str&gt;
    &lt;str name="weightField"&gt;price&lt;/str&gt;
    &lt;str name="suggestAnalyzerFieldType"&gt;string&lt;/str&gt;
  &lt;/lst&gt;
  &lt;lst name="suggester"&gt;
    &lt;str name="name"&gt;altSuggester&lt;/str&gt;
    &lt;str name="dictionaryImpl"&gt;DocumentExpressionDictionaryFactory&lt;/str&gt;
    &lt;str name="lookupImpl"&gt;FuzzyLookupFactory&lt;/str&gt;
    &lt;str name="field"&gt;product_name&lt;/str&gt;
    &lt;str name="weightExpression"&gt;((price * 2) + ln(popularity))&lt;/str&gt;
    &lt;str name="sortField"&gt;weight&lt;/str&gt;
    &lt;str name="sortField"&gt;price&lt;/str&gt;
    &lt;str name="storeDir"&gt;suggest_fuzzy_doc_expr_dict&lt;/str&gt;
    &lt;str name="suggestAnalyzerFieldType"&gt;text_en&lt;/str&gt;
  &lt;/lst&gt;
&lt;/searchComponent&gt;
```
在查询中使用这些 Suggesters 时，您可以在请求中定义多个 suggest.dictionary 参数，引用在搜索组件定义中为每个 Suggester 指定的名称。响应将包括每个提示的章节中的 term。有关示例请求和响应，请参见下面的示例使用部分。
      
  

## 添加 Suggest 请求处理程序<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#adding-the-suggest-request-handler"/>

添加搜索组件后，必须添加一个请求处理程序到 solrconfig.xml 中。这个请求处理程序与任何其他请求处理程序一样工作，并允许您为服务建议请求配置默认参数。请求处理程序定义必须包含之前定义的“建议”搜索组件。
      
  
```
&lt;requestHandler name="/suggest" class="solr.SearchHandler" startup="lazy"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="suggest"&gt;true&lt;/str&gt;
    &lt;str name="suggest.count"&gt;10&lt;/str&gt;
  &lt;/lst&gt;
  &lt;arr name="components"&gt;
    &lt;str&gt;suggest&lt;/str&gt;
  &lt;/arr&gt;
&lt;/requestHandler&gt;
```

### 建议请求处理程序参数<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#suggest-request-handler-parameters"/>

以下参数允许您为建议请求处理程序设置默认值：  
- suggest=true  
    
        这个参数应该总是<code>true</code>，因为我们总是想运行提交给这个处理程序的查询的 Suggester。  
    
- suggest.dictionary  
   
        在搜索组件中配置的字典组件的名称。这是一个强制参数。它可以在请求处理程序中设置，或者在查询时作为参数发送。  
    
- suggest.q  
    
        用于建议查找的查询。  
    
- suggest.count  
    
        指定 Solr 返回的建议数量。  
    
- suggest.cfq  
    
        上下文过滤器查询，用于根据上下文字段筛选建议 (如果 suggester 支持)。  
    
- suggest.build  
    
        如果<code>true</code>，它会建立建议索引。这可能只对最初的请求有用；您可能不想在每个请求上建立字典，特别是在生产系统中。如果您想保持您的字典是最新的，您应该使用<code>buildOnCommit</code>或<code>buildOnOptimize</code>参数搜索组件。  
    
- suggest.reload  
    
        如果<code>true</code>它将重新加载建议索引。  
    
- suggest.buildAll  
   
        如果<code>true</code>，它会建立所有的建议索引。  
    
- suggest.reloadAll  
    
        如果<code>true</code>它将重新加载所有的建议索引。  
    

这些属性也可以在查询时被覆盖，或者根本不在请求处理器中设置，并且总是在查询时发送。  
上下文过滤：上下文过滤（<code>suggest.cfq</code>）目前只支持<code>AnalyzingInfixLookupFactory</code>和<code>BlendedInfixLookupFactory</code>，并且只有在<code>Document*Dictionary</code>支持的情况下。所有其他的实现将返回未经过滤的匹配，就好像没有请求过滤一样。  

## 使用<span style="font-family: inherit; font-size: 16px; font-weight: 600;">示例</span>

## <a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#example-usages"/>

### 通过权重获取建议<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#get-suggestions-with-weights"/>

这是使用单个字典和单个 Solr 核心的基本建议。  
示例查询：  
```
http://localhost:8983/solr/techproducts/suggest?suggest=true&amp;suggest.build=true&amp;suggest.dictionary=mySuggester&amp;suggest.q=elec
```
在这个例子中，我们简单地使用 suggest.q 参数请求了字符串 “elec”，并要求建立字典 suggest.build（但是，注意，您可能不想在每个查询上建立索引 - 那么如果你经常更换文件，则应该使用 buildOnCommit 或者buildOnOptimize ）。
      
  
响应示例：  
```
{
  "responseHeader": {
    "status": 0,
    "QTime": 35
  },
  "command": "build",
  "suggest": {
    "mySuggester": {
      "elec": {
        "numFound": 3,
        "suggestions": [
          {
            "term": "electronics and computer1",
            "weight": 2199,
            "payload": ""
          },
          {
            "term": "electronics",
            "weight": 649,
            "payload": ""
          },
          {
            "term": "electronics and stuff2",
            "weight": 279,
            "payload": ""
          }
        ]
      }
    }
  }
}
```

### 使用多个词典<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#using-multiple-dictionaries"/>

如果您已经定义了多个字典，您可以在查询中使用它们。  
示例查询：  
```
http://localhost:8983/solr/techproducts/suggest?suggest=true&amp;suggest.dictionary=mySuggester&amp;suggest.dictionary=altSuggester&amp;suggest.q=elec
```
在这个例子中，我们发送了字符串 'elec' 作为 suggest.q 参数，并命名了两个要使用的 suggest.dictionary 定义。  
响应示例：  
```
{
  "responseHeader": {
    "status": 0,
    "QTime": 3
  },
  "suggest": {
    "mySuggester": {
      "elec": {
        "numFound": 1,
        "suggestions": [
          {
            "term": "electronics and computer1",
            "weight": 100,
            "payload": ""
          }
        ]
      }
    },
    "altSuggester": {
      "elec": {
        "numFound": 1,
        "suggestions": [
          {
            "term": "electronics and computer1",
            "weight": 10,
            "payload": ""
          }
        ]
      }
    }
  }
}
```

### 上下文过滤<a href="http://lucene.apache.org/solr/guide/7_0/suggester.html#context-filtering"/>

上下文过滤允许您通过单独的上下文字段（如类别、部门或任何其他标记）来过滤建议。AnalyzingInfixLookupFactory 与 BlendedInfixLookupFactory 目前支持此功能，当支持 DocumentDictionaryFactory。
      
  
添加 contextField 到您的 suggester 配置中。这个例子将建议名称，并允许按类别过滤：  
在 solrconfig.xml 中：  
```
&lt;searchComponent name="suggest" class="solr.SuggestComponent"&gt;
  &lt;lst name="suggester"&gt;
    &lt;str name="name"&gt;mySuggester&lt;/str&gt;
    &lt;str name="lookupImpl"&gt;AnalyzingInfixLookupFactory&lt;/str&gt;
    &lt;str name="dictionaryImpl"&gt;DocumentDictionaryFactory&lt;/str&gt;
    &lt;str name="field"&gt;name&lt;/str&gt;
    &lt;str name="weightField"&gt;price&lt;/str&gt;
    &lt;str name="contextField"&gt;cat&lt;/str&gt;
    &lt;str name="suggestAnalyzerFieldType"&gt;string&lt;/str&gt;
    &lt;str name="buildOnStartup"&gt;false&lt;/str&gt;
  &lt;/lst&gt;
&lt;/searchComponent&gt;
```
示例-上下文过滤建议查询：  
```
http://localhost:8983/solr/techproducts/suggest?suggest=true&amp;suggest.build=true&amp;suggest.dictionary=mySuggester&amp;suggest.q=c&amp;suggest.cfq=memory
```
suggester 只会带回标记为“cat = memory”的产品的建议。  
