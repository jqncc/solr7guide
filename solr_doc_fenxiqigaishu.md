## Solr分析器概述 
<div class="content-intro view-box ">分析器检查字段的文本并生成令牌流。  
  
Solr 分析器被指定为 schema.xml 配置文件中的&lt;fieldType&gt;元素的子元素（在与 solrconfig. xml 相同的 conf/ 目录中）。  
在正常使用情况下，只有类型为 solr.TextField 的字段将指定一个分析器。配置分析器的最简单的方法是使用单个 &lt;analyzer&gt; 元素，它的类属性是一个完全限定的Java 类名。命名的类必须派生自 org.apache.lucene.analysis.Analyzer。例如：  
```
&lt;fieldType name="nametext" class="solr.TextField"&gt;
  &lt;analyzer class="org.apache.lucene.analysis.core.WhitespaceAnalyzer"/&gt;
&lt;/fieldType&gt;
```
在这种情况下，单个类 WhitespaceAnalyzer 负责分析指定文本字段的内容并发出相应的令牌。对于简单的情况，如简单的英文散文，这样的单个分析器类可能就足够了。但是通常需要对字段内容进行更复杂的分析。  
  
即使是最复杂的分析要求，通常也可以分解为一系列离散的、相对简单的处理步骤。正如你很快就会发现的那样，Solr 发行版提供了大量的标记器和过滤器，覆盖了你可能遇到的大多数场景。建立一个分析器链非常简单，您可以指定一个简单的&lt; analyzer &gt;元素（无类属性），使用子元素命名标记器和过滤器的工厂类以使用，按照您希望它们运行的​​顺序。  
  
例如：  
```
&lt;fieldType name="nametext" class="solr.TextField"&gt;
  &lt;analyzer&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.StandardFilterFactory"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
    &lt;filter class="solr.StopFilterFactory"/&gt;
    &lt;filter class="solr.EnglishPorterFilterFactory"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
请注意，org.apache.solr.analysis 包中的类可能在这里用简写的 solr. 前缀来引用。  
在这种情况下，&lt;analyzer&gt; 元素上没有指定分析器类。相反，一系列更专业化的类连接在一起，共同作为该字段的分析器。该字段的文本被传递给list（solr.StandardTokenizerFactory）中的第一个项目，而从最后一个（solr.EnglishPorterFilterFactory）中出现的标记是用于对任何使用 "nametext" fieldType 的字段进行索引或查询的术语。  
字段值与索引术语：分析器的输出会影响给定字段中索引的术语 (以及分析对这些字段的查询时使用的术语)，但不会影响字段的存储值。例如: “Brown Cow”分成两个索引词 “brown” 和 “cow”，但存储的值仍将是一个字符串: “Brown Cow”。  
  
## <span style="font-family: inherit; font-size: 16px; font-weight: 600;">分析阶段</span>

## <a href="http://lucene.apache.org/solr/guide/7_0/analyzers.html#analysis-phases"/>

分析发生在两种情况下：在索引的时候，当一个字段被创建时，分析得到的令牌流将被添加到一个索引中，并为该字段定义一组术语（包括位置、大小等）。在查询时间，分析正在搜索的值，并将结果的条件与存储在字段索引中的条件进行匹配。  
  
在很多情况下，对两个阶段都应该进行相同的分析。例如，当您想要查询精确的字符串匹配时，这可能是不区分大小写的。在其他情况下，您可能希望在索引期间应用略有不同的分析步骤，而不是在查询时使用的分析步骤。  
如果您为 &lt;analyzer&gt; 字段类型提供了一个简单的定义（如上例所示），那么它将用于索引和查询。如果您想要为每个阶段使用不同的分析器，则可以包含两个与 type 属性区分的 &lt;analyzer&gt; 定义。例如：  
```
&lt;fieldType name="nametext" class="solr.TextField"&gt;
  &lt;analyzer type="index"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
    &lt;filter class="solr.KeepWordFilterFactory" words="keepwords.txt"/&gt;
    &lt;filter class="solr.SynonymFilterFactory" synonyms="syns.txt"/&gt;
  &lt;/analyzer&gt;
  &lt;analyzer type="query"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
在这个理论性的例子中，在索引时，文本被标记化，标记被设置为小写，任何未列出的 keepwords.txt 都被丢弃，而保留的那些将被映射到文件 syns.txt 中的同义词规则所定义的替代值。这基本上是从一组受限制的可能值生成索引，然后将它们规范化为可能甚至不会出现在原始文本中的值。  
  
在查询时，唯一发生的规范化是将查询条件转换为小写。在索引时发生的筛选和映射步骤不适用于查询条件。在这个例子中，查询必须非常精确，仅使用在索引时存储的规范化术语。  

### 分析多期扩展<a href="http://lucene.apache.org/solr/guide/7_0/analyzers.html#analysis-for-multi-term-expansion"/>

在某些类型的查询中（即：前缀、通配符、正则表达式等等），用户提供的输入不是用于分析的自然语言。诸如同义词或停用词过滤之类的东西在这些类型的查询中不起作用。  
  
可以在这些类型的查询（（如 Lowercasing 或 Normalizing Factories））中工作的分析工厂被称为 MultiTermAwareComponents。当 Solr 需要对导致 Multi-Term 扩展的查询执行分析时，只使用在 query 分析器中使用的 MultiTermAwareComponents，不跳过 Multi-Term 的 Factory 将被跳过。  
对于大多数使用情况，这提供了最好的行为，但是如果希望绝对控制对这些类型查询执行的分析，则可以明确定义一个要使用的 multiterm 分析器，如下例所示：  
```
&lt;fieldType name="nametext" class="solr.TextField"&gt;
  &lt;analyzer type="index"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
    &lt;filter class="solr.KeepWordFilterFactory" words="keepwords.txt"/&gt;
    &lt;filter class="solr.SynonymFilterFactory" synonyms="syns.txt"/&gt;
  &lt;/analyzer&gt;
  &lt;analyzer type="query"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
  &lt;/analyzer&gt;
  &lt;!-- No analysis at all when doing queries that involved Multi-Term expansion --&gt;
  &lt;analyzer type="multiterm"&gt;
    &lt;tokenizer class="solr.KeywordTokenizerFactory" /&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
