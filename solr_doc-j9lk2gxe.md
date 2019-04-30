## Solr使用之拼写检查（SpellCheck） 
<div class="content-intro view-box ">SpellCheck（拼写检查）组件旨在提供基于其他相似 term 的内嵌查询建议。
      
  
这些建议的基础可以是 Solr 中的字段、外部创建的文本文件或其他 Lucene 索引中的字段的 term。  

## 配置 SpellCheckComponent<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#configuring-the-spellcheckcomponent"/>


### 在 solrconfig.xml 中定义拼写检查<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#define-spell-check-in-solrconfig-xml"/>

第一步是在 solrconfig.xml 中指定 term 的来源。在 Solr 中有三种拼写检查方法，在下面讨论。
      
  

#### IndexBasedSpellChecker<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#indexbasedspellchecker"/>

IndexBasedSpellChecker 使用 Solr 索引作为用于拼写检查并行索引的基础。它要求定义一个字段作为索引 term 的基础；通常的做法是将 term 从某些字段 （如 title，body 等），复制到为拼写检查而创建的另一个字段中。下面是一个使用 IndexBasedSpellChecker 配置 solrconfig. xml 的简单示例：  
```
&lt;searchComponent name="spellcheck" class="solr.SpellCheckComponent"&gt;
  &lt;lst name="spellchecker"&gt;
    &lt;str name="classname"&gt;solr.IndexBasedSpellChecker&lt;/str&gt;
    &lt;str name="spellcheckIndexDir"&gt;./spellchecker&lt;/str&gt;
    &lt;str name="field"&gt;content&lt;/str&gt;
    &lt;str name="buildOnCommit"&gt;true&lt;/str&gt;
    &lt;!-- optional elements with defaults
    &lt;str name="distanceMeasure"&gt;org.apache.lucene.search.spell.LevensteinDistance&lt;/str&gt;
    &lt;str name="accuracy"&gt;0.5&lt;/str&gt;
    --&gt;
 &lt;/lst&gt;
&lt;/searchComponent&gt;
```

第一个元素定义了要使用 solr.SpellCheckComponent 的 searchComponent。这里 classname 是 SpellCheckComponent 的具体实现
      
  
：solr.IndexBasedSpellChecker。定义 classname 是可选的；如果没有定义，它将默认为：IndexBasedSpellChecker。  
spellcheckIndexDir 定义了保持所述拼写检查索引的目录的位置，而字段定义了用于拼写检查 term 的源字段（在架构中定义）。为拼写检查索引选择一个字段时，最好避免经过大量处理的字段以获得更准确的结果。如果该字段具有处理同义词或词干的许多单词变体，则除了更多有效的拼写数据之外，将使用这些变体创建词典。  
最后，buildOnCommit 定义是否在每次提交时都建立拼写检查索引（也就是每次将新文档添加到索引时）。这是可选的，如果您愿意的话可以将其设置为 false，也可以省略它。  

#### DirectSolrSpellChecker<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#directsolrspellchecker"/>

在 DirectSolrSpellChecker 使用 Solr 索引中的 term，而没有构建类似 IndexBasedSpellChecker 的并行索引。这个拼写检查器具有不必定期建立的好处，这意味着 term 始终与索引中的term 保持一致。下述例子表示了如何在 solrconfig.xml 中配置：  
```
&lt;searchComponent name="spellcheck" class="solr.SpellCheckComponent"&gt;
  &lt;lst name="spellchecker"&gt;
    &lt;str name="name"&gt;default&lt;/str&gt;
    &lt;str name="field"&gt;name&lt;/str&gt;
    &lt;str name="classname"&gt;solr.DirectSolrSpellChecker&lt;/str&gt;
    &lt;str name="distanceMeasure"&gt;internal&lt;/str&gt;
    &lt;float name="accuracy"&gt;0.5&lt;/float&gt;
    &lt;int name="maxEdits"&gt;2&lt;/int&gt;
    &lt;int name="minPrefix"&gt;1&lt;/int&gt;
    &lt;int name="maxInspections"&gt;5&lt;/int&gt;
    &lt;int name="minQueryLength"&gt;4&lt;/int&gt;
    &lt;float name="maxQueryFrequency"&gt;0.01&lt;/float&gt;
    &lt;float name="thresholdTokenFrequency"&gt;.01&lt;/float&gt;
  &lt;/lst&gt;
&lt;/searchComponent&gt;
```

当选择 field 以查询这个拼写检查器时，您需要对其进行相对较少的分析 (特别是对词干分析)。请注意，您需要指定一个字段中使用您的建议，比如 IndexBasedSpellChecker，您可能希望将数据从像 title，body 等等的字段中复制到专门提供拼写建议的字段中。
      
  
许多参数与这个拼写检查器应该如何查询索引的词条建议有关。该 distanceMeasure 定义了在拼写检查查询期间使用的跃点数。“内部”值使用默认的 Levenshtein 度量，它与其他拼写检查器实现使用的度量标准相同。  
由于此拼写检查器正在查询主索引，因此您可能需要限制查询索引的频率，以避免与用户查询发生任何性能冲突。该 accuracy 设置定义有效建议的阈值，同时 maxEdits 定义允许的 term 更改次数。由于大多数拼写错误都只有一个字母，所以将其设置为1会减少可能的建议的数量（默认值为2）。该值只能是1或2. minPrefix 定义术语应共享的最少字符数。将其设置为1意味着拼写建议将全部以相同的字母开始。  
该 maxInspections 参数定义在返回结果之前要审阅的可能匹配项的最大数目；默认值是 5。minQueryLength 定义在提供建议之前查询中必须有多少个字符；默认是 4。  
首先，拼写检查器通过在索引中查找来分析传入的查询词。只有在索引中不存在的查询词或者非常少见的词（低于 maxQueryFrequency）被认为是拼写错误的，用于查找建议。比maxQueryFrequency 绕过拼写检查器更频繁的词不变。在找到每个拼写错误的单词的建议之后，将其过滤为具有足够频率的 thresholdTokenFrequency 边界值。这些参数（maxQueryFrequency 和 thresholdTokenFrequency）可以是百分比（如.01或1％）或绝对值（如4）。  

#### FileBasedSpellChecker<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#filebasedspellchecker"/>

FileBasedSpellChecker 使用外部文件作为拼写字典。如果将 Solr 用作拼写服务器，或者拼写建议不需要基于索引中的实际 term，这会很有用。在solrconfig.xml，您可以这样定义searchComponent：  
```
&lt;searchComponent name="spellcheck" class="solr.SpellCheckComponent"&gt;
  &lt;lst name="spellchecker"&gt;
    &lt;str name="classname"&gt;solr.FileBasedSpellChecker&lt;/str&gt;
    &lt;str name="name"&gt;file&lt;/str&gt;
    &lt;str name="sourceLocation"&gt;spellings.txt&lt;/str&gt;
    &lt;str name="characterEncoding"&gt;UTF-8&lt;/str&gt;
    &lt;str name="spellcheckIndexDir"&gt;./spellcheckerFile&lt;/str&gt;
    &lt;!-- optional elements with defaults
    &lt;str name="distanceMeasure"&gt;org.apache.lucene.search.spell.LevensteinDistance&lt;/str&gt;
    &lt;str name="accuracy"&gt;0.5&lt;/str&gt;
    --&gt;
 &lt;/lst&gt;
&lt;/searchComponent&gt;
```

这里的区别是使用 sourceLocation 来定义 term 文件的位置和使用 characterEncoding 来定义 term 文件的编码。
      
  
在前面的示例中，名称用于命名检查的这个特定定义。多个定义可以共存于单个 solrconfig. xml 中，名称有助于区分它们。如果只定义一个拼写检查器，则不需要名称。
      
  

#### WordBreakSolrSpellChecker<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#wordbreaksolrspellchecker"/>

WordBreakSolrSpellChecker 通过将相邻查询 term 和 将 term 分解成多个单词来提供建议。这是一个 SpellCheckComponent 的增强，利用 Lucene 的 WordBreakSpellChecker。它可以在不使用 shingle-based 字典的情况下检测由错位的空白产生的拼写错误，并且提供对单词中断错误的整理支持，包括用户在同一个查询中混合了单词拼写错误和单词中断错误的情况。它还提供碎片支持。
      
  
以下是如何在 solrconfig 中配置它的示例：  
```
&lt;searchComponent name="spellcheck" class="solr.SpellCheckComponent"&gt;
  &lt;lst name="spellchecker"&gt;
    &lt;str name="name"&gt;wordbreak&lt;/str&gt;
    &lt;str name="classname"&gt;solr.WordBreakSolrSpellChecker&lt;/str&gt;
    &lt;str name="field"&gt;lowerfilt&lt;/str&gt;
    &lt;str name="combineWords"&gt;true&lt;/str&gt;
    &lt;str name="breakWords"&gt;true&lt;/str&gt;
    &lt;int name="maxChanges"&gt;10&lt;/int&gt;
  &lt;/lst&gt;
&lt;/searchComponent&gt;
```

一些参数将在讨论其他拼写检查时熟悉，如 name、classname 和 field。这个拼写检查器的新功能是：combineWords，它定义了在字典搜索中是否应合并单词（默认为true）；breakWords，它定义在字典搜索期间单词是否应该被中断（默认为 true）；maxChanges，它是一个整数，它定义了拼写检查器应该检查对索引的排序可能性的次数（默认值是10）。
      
  
拼写检查器可以配置一个传统的检查器（即：DirectSolrSpellChecker）。结果是结合在一起的，整理可以包含来自两个 spellcheckers 的更正组合。  

### 将它添加到请求处理程序<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#add-it-to-a-request-handler"/>

查询将被发送到 RequestHandler。如果每个请求都应该生成一个建议，那么您会添加以下内容到您正在使用的 requestHandler ：  
```
&lt;str name="spellcheck"&gt;true&lt;/str&gt;
```

其中一个可能的参数是使用的 spellcheck.dictionary ，并且可以定义倍数。使用多个词典，查询所有指定的词典，并将结果交错。排序规则是使用来自不同拼写检查程序的组合创建的，要注意在同一排序规则中不会发生多个重叠的更正。
      
  
下面是一个具有多个字典的示例：  
```
&lt;requestHandler name="spellCheckWithWordbreak" class="org.apache.solr.handler.component.SearchHandler"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="spellcheck.dictionary"&gt;default&lt;/str&gt;
    &lt;str name="spellcheck.dictionary"&gt;wordbreak&lt;/str&gt;
    &lt;str name="spellcheck.count"&gt;20&lt;/str&gt;
  &lt;/lst&gt;
  &lt;arr name="last-components"&gt;
    &lt;str&gt;spellcheck&lt;/str&gt;
  &lt;/arr&gt;
&lt;/requestHandler&gt;
```


## 拼写检查参数<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#spell-check-parameters"/>

拼写检查组件接受下面描述的参数：  
- spellcheck 参数  

   
        此参数为请求打开拼写检查建议。如果<code>true</code>，则会生成拼写建议。如果需要拼写检查，则这是必需的。  
    
- spellcheck.q 或者 q 参数  

    
        该参数指定拼写检查的查询。  
        
            如果<code>spellcheck.q</code>被定义，则被使用；否则使用原始输入查询。该<code>spellcheck.q</code>参数旨在成为原始查询，减去字段名称，提升等额外的标记。如指定了这个 q 参数，那么这个<code>SpellingQueryConverter</code>类就被用来解析成标记；否则使用[<code>WhitespaceTokenizer</code>](http://lucene.apache.org/solr/guide/7_0/tokenizers.html#white-space-tokenizer)。  
          
        
            选择使用哪一个取决于应用程序。从本质上讲，如果你的应用程序中有一个拼写“准备好”的版本，那么最好使用<code>spellcheck.q</code>。否则，如果您只是想要 Solr 做这个工作，使用<code>q</code>参数。  
          
  
        <code>SpellingQueryConverter</code>类不使用非ASCII字符妥善处理。在这种情况下，您必须使用<code>spellcheck.q</code>或实现您自己的QueryConverter  
    

- spellcheck.build 参数  

   
        如果设置为<code>true</code>，则此参数将创建用于拼写检查的字典。在典型的搜索应用程序中，您需要在使用拼写检查之前构建字典。但是，并不总是需要先建立一个字典。例如，您可以将拼写检查器配置为使用已存在的字典。  
        
            字典将需要一些时间来建立，所以这个参数不应该与每个请求一起发送。  
          
    
- spellcheck.reload 参数  

    
        如果设置为<code>true</code>，则此参数将重新加载拼写检查程序。结果取决于执行<code>SolrSpellChecker.reload()</code>。在典型的实现中，重新加载拼写检查程序意味着重新加载字典。  
    
- spellcheck.count 参数  

    
        此参数指定拼写检查器应为某个术语返回的最大建议数。如果未设置此参数，则该值默认为<code>1</code>。如果参数已设置但未分配一个数字，则该值默认为<code>5</code>。如果参数设置为正整数，则该数字将成为拼写检查程序返回的建议的最大数量。  
    
- spellcheck.onlyMorePopular 参数  

    
        如果为<code>true</code>，Solr 将返回比现有查询更有可能导致查询命中次数的建议。请注意，即使索引中存在给定查询词并认为“正确”，也会返回更受欢迎的建议。  
    
- spellcheck.maxResultsForSuggest 参数  

   
        例如，如果设置为<code>5</code>，并且用户的查询返回5个或更少的结果，则拼写检查程序将报告“correctSpelled = false”，并提供建议（以及请求时的排序规则）。设置这个大于零对于创建“did-you-mean?”的查询（对返回低数量的命中）很有用。  
    
- spellcheck.alternativeTermCount 参数  
    
        定义索引或字典中存在的每个查询 term 的返回建议数。据推测，用户会希望对 docFrequency&gt; 0 的单词提出更少的建议。此外，设置此值可启用上下文相关的拼写建议。  
    
- spellcheck.extendedResults 参数  

    
        如果为<code>true</code>此参数导致 Solr 返回有关拼写检查结果的附加信息，例如 index（<code>origFreq</code>）中每个原始词语的频率以及 index（frequency）中每个建议的频率。请注意，这个结果格式与非扩展格式不同，因为返回的单词建议实际上是一个列表数组，其中每个列表包含建议的术语及其频率。
              
          
    
- spellcheck.collate 参数  

   
        如果为<code>true</code>此参数指示 Solr 对每个标记（如果存在）采取最佳建议，并根据建议构建新的查询。  
        
            例如，如果输入查询是“jawa class lording”，而“jawa”的最佳建议是“java”，“lording”是“loading”，那么结果归类为“java class loading”。  
          
        
            该<code>spellcheck.collate</code>参数只返回确保在 re-queried 时会导致命中的排序规则, 即使在应用原始的<code>fq</code>参数时也是如此。每个查询有多个更正时，这是特别有用的。  
          
   
        这仅返回要使用的查询。它实际上并不运行建议的查询。
              
          
    

- spellcheck.maxCollations 参数  

  
        要返回的最大排序数。默认是<code>1</code>。如果<code>spellcheck.collate</code>为 false，则忽略此参数。  
    
- spellcheck.maxCollationTries 参数  

    
        此参数指定 Solr 在放弃之前尝试进行排序的可能性数目。值越低，性能越好。找到可以返回结果的排序规则可能需要更高的值。默认值是<code>0</code>，这相当于不检查排序规则。如果<code>spellcheck.collate</code>为 false，则忽略此参数。  
    
- spellcheck.maxCollationEvaluations 参数  
    
        此参数指定在确定要对索引进行测试的排序规则候选项之前，要对其进行排序和计算的单词更正组合的最大数目。如果用户输入含有许多拼写错误的单词的查询，这是一个性能安全网。默认值是<code>10000</code>组合，这在大多数情况下应该可以正常工作。  
    
- spellcheck.collateExtendedResults  

    
        如果为<code>true</code>此参数返回扩展的响应格式，详细说明 Solr 找到的排序规则。默认值是<code>false</code>，如果<code>spellcheck.collate</code>为 false，则忽略此错误。  
    
- spellcheck.collateMaxCollectDocs 函数  

    
        此参数指定在根据索引测试潜在归类时应收集的最大文档数量。值<code>0</code>表示应该收集所有文档，从而产生确切的命中数。否则，在不需要精确的命中计数的情况下，作为性能优化提供了一种估计--指定的值越高，估计就越精确。
              
          
  
        
            此参数的默认值是<code>0</code>，但是当<code>spellcheck.collateExtendedResults</code>为 false 时，总是使用优化，像指定了<code>1</code>一样。  
          
    
- spellcheck.collateParam.* Prefix  
    
        此参数前缀可用于指定在内部验证归类查询时希望检查使用的任何其他参数。例如，即使您的常规搜索结果允许通过类似的参数松散地匹配一个或多个查询词语如：<code>q.op=OR</code>和<code>mm=20%</code>您也可以指定重写参数，例如<code>spellcheck.collateParam.q.op=AND&amp;spellcheck.collateParam.mm=100%</code>要求需要只有在至少一个文档中找到的所有单词的排序规则才会被返回。
              
          
    
- spellcheck.dictionary 参数  

   
        此参数使 Solr 使用参数的参数中指定的字典。默认设置是<code>default</code>。该参数可用于根据请求调用特定的拼写检查器。  
    
- spellcheck.accuracy 参数  

    
        指定拼写检查实现使用的准确度值，以确定结果是否值得。该值是0到1之间的浮点数。默认为<code>Float.MIN_VALUE</code>。  
    
- spellcheck.&lt;DICT_NAME&gt;.key 参数  

   
        指定用于处理给定字典的实现的键/值对。传递的值就是<code>key=value</code>（<code>spellcheck.&lt;DICT_NAME&gt;.</code>被剥离）。  
        
            例如，给定一个字典调用<code>foo</code>，<code>spellcheck.foo.myKey=myValue</code>会导致<code>myKey=myValue</code>被传递到处理字典<code>foo</code>的实现。  
          
    


### 拼写检查示例<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#spell-check-example"/>

使用 Solr 的 bin/solr -e techproducts 示例，这个查询显示了一个简单的请求的结果，该请求使用 spellcheck.q 参数定义了一个查询，并强制排序规则要求所有输入条件必须匹配：
      
  
```
http://localhost:8983/solr/techproducts/spell?df=text&amp;spellcheck.q=delll+ultra+sharp&amp;spellcheck=true&amp;spellcheck.collateParam.q.op=AND
```

结果如下：  
```
&lt;lst name="spellcheck"&gt;
  &lt;lst name="suggestions"&gt;
    &lt;lst name="delll"&gt;
      &lt;int name="numFound"&gt;1&lt;/int&gt;
      &lt;int name="startOffset"&gt;0&lt;/int&gt;
      &lt;int name="endOffset"&gt;5&lt;/int&gt;
      &lt;int name="origFreq"&gt;0&lt;/int&gt;
      &lt;arr name="suggestion"&gt;
        &lt;lst&gt;
          &lt;str name="word"&gt;dell&lt;/str&gt;
          &lt;int name="freq"&gt;1&lt;/int&gt;
        &lt;/lst&gt;
      &lt;/arr&gt;
    &lt;/lst&gt;
    &lt;lst name="ultra sharp"&gt;
      &lt;int name="numFound"&gt;1&lt;/int&gt;
      &lt;int name="startOffset"&gt;6&lt;/int&gt;
      &lt;int name="endOffset"&gt;17&lt;/int&gt;
      &lt;int name="origFreq"&gt;0&lt;/int&gt;
      &lt;arr name="suggestion"&gt;
        &lt;lst&gt;
          &lt;str name="word"&gt;ultrasharp&lt;/str&gt;
          &lt;int name="freq"&gt;1&lt;/int&gt;
        &lt;/lst&gt;
      &lt;/arr&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
  &lt;bool name="correctlySpelled"&gt;false&lt;/bool&gt;
  &lt;lst name="collations"&gt;
    &lt;lst name="collation"&gt;
      &lt;str name="collationQuery"&gt;dell ultrasharp&lt;/str&gt;
      &lt;int name="hits"&gt;1&lt;/int&gt;
      &lt;lst name="misspellingsAndCorrections"&gt;
        &lt;str name="delll"&gt;dell&lt;/str&gt;
        &lt;str name="ultra sharp"&gt;ultrasharp&lt;/str&gt;
      &lt;/lst&gt;
    &lt;/lst&gt;
  &lt;/lst&gt;
&lt;/lst&gt;
```


## 分布式拼写检查<a href="http://lucene.apache.org/solr/guide/7_0/spell-checking.html#distributed-spellcheck"/>

该 SpellCheckComponent 还支持分布式索引拼写检查。如果您在除 “/ select” 以外的请求处理程序上使用 SpellCheckComponent，则必须提供以下两个参数：  
- shards 参数  

    
        指定分布式索引配置中的分片。有关分布式索引的更多信息，请参见使用索引分片的分布式搜索  
    
- shards.qt 参数  

    
        指定 Solr 用于请求分片的请求处理程序。<code>/select</code>请求处理程序不需要此参数。  
    

例如：  
```
http://localhost:8983/solr/techproducts/spell?spellcheck=true&amp;spellcheck.build=true&amp;spellcheck.q=toyata&amp;shards.qt=/spell&amp;shards=solr-shard1:8983/solr/techproducts,solr-shard2:8983/solr/techproducts
```

在对 SpellCheckComponent 的分发请求的情况下，即使 spellcheck.count 参数值小于五，分片请求至少五个建议。一旦建议被收集，他们由被配置的距离措施排列（默认为 Levenstein 距离），然后按聚合频率。  
