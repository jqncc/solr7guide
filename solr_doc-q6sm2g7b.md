## 什么是Solr过滤器 
<div class="content-intro view-box ">与分词一样，Solr 过滤器消耗输入并产生一个令牌流。过滤器也来自于 org.apache.lucene.analysis.TokenStream。与分词不同，过滤器的输入是另一个TokenStream。过滤器的工作通常比 tokenizer 更容易，因为在大多数情况下，过滤器会依次查看流中的每个标记，并决定是否将其传递、替换或丢弃。  
  
过滤器也可以通过预先考虑多个令牌来进行更复杂的分析，尽管这种情况不常见。这种过滤器的一个假设的用法可能是规范化将被标记为两个单词的状态名称。例如，单个标记 “california” 将被替换为 “CA”，而令牌对 “rhode” 后面跟着 “island” 将变成单个令牌 “RI”。  
因为过滤器消耗一个 TokenStream 并产生一个新的过滤器 TokenStream，它们可以无限地连续链接在一起。链中的每个过滤器反过来处理由其前置生成的令牌。因此，指定过滤器的顺序非常重要。通常，首先进行最常规的过滤，稍后的过滤阶段更专业化。  
```
&lt;fieldType name="text" class="solr.TextField"&gt;
  &lt;analyzer&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.StandardFilterFactory"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
    &lt;filter class="solr.EnglishPorterFilterFactory"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
这个例子从 Solr 的标准 tokenizer 开始，它将字段的文本分解为令牌。然后，这些令牌通过 Solr 的标准过滤器，从首字母缩略词中删除点，并执行一些其他的常见操作。所有的令牌都被设置为小写，这将有利于在查询时不区分大小写的匹配。  
上例中的最后一个过滤器是使用 Porter 干扰算法的干涉过滤器。词干分析器基本上是一套映射规则，将单词的各种形式映射回它们派生出来的基础词或词干。例如，在英语中，“hugs”，“hugging” 和 “hugged” 这两个词都是词干 “hug” 的所有形式。词干将用 “hug” 来代替所有这些词语，这将被索引。这意味着对 “hug” 的查询将匹配术语 “hugged”，而不是 “huge”。  
相反，将词干应用于查询条件将允许包含非词干术语（例如“hugging”）的查询匹配具有相同词干的不同变体（例如“hugged”）的文档。这是可行的，因为索引器和查询都将映射到相同的词干（“hug”）。  
词干明显地是非常特定于语言的。Solr 包含了几个基于 Porter 干扰算法的由 Snowball 生成器创建的特定于语言的干预器。通用的 Snowball Porter Stemmer Filter可用于配置这些语言分析器。Solr 还为 English Snowball 词干提供了一个便利的包装。还有几种专门为非英语语言设计的词干。这些词干在语言分析中有描述。  
