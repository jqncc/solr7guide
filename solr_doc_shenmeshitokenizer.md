## 什么是Tokenizer 
<div class="content-intro view-box ">Tokenizer 的工作是将文本流分解为令牌，其中每个令牌（通常）是文本中字符的子序列。分析器知道它配置的字段，但 tokenizer 不是。Tokenizers 从字符流（Reader）中读取并生成一系列令牌对象（TokenStream）。  
  
输入流中的字符可能被丢弃，如空格或其他分隔符。也可以添加或替换它们，例如将别名或缩写映射到规范化的窗体。令牌包含除文本值之外的各种元数据，例如字段中令牌出现的位置。由于 Tokenizer 可能会产生与输入文本不一致的标记，因此不应假定该标记的文本与字段中出现的文本相同，或者其长度与原始文本相同。也可能有多个令牌具有相同的位置或引用原始文本中的相同偏移量。请记住，如果您使用令牌元数据来突出显示字段文本中的搜索结果。  
```
&lt;fieldType name="text" class="solr.TextField"&gt;
  &lt;analyzer&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
在 tokenizer 元素中命名的类不是实际的 tokenizer，而是实现 TokenizerFactory API 的类。这个工厂类将被调用来根据需要创建新的 tokenizer 实例。由工厂创建的对象必须来自于 Tokenizer，这表示它们产生令牌序列。如果令牌生成器产生可以按原样使用的令牌，则它可能是分析器的唯一组件。否则，tokenizer 的输出标记将作为流水线中第一个过滤器阶段的输入。  
  
TypeTokenFilterFactory 可用与创建 TypeTokenFilter 根据在 factory.getStopTypes 中设置的 TypeAttribute。  
有关可用 TokenFilters 的完整列表，请参阅 Tokenizers 部分。  

## 何时使用 CharFilter 与 TokenFilter<a href="http://lucene.apache.org/solr/guide/7_0/about-tokenizers.html#when-to-use-a-charfilter-vs-a-tokenfilter"/>

有几个 CharFilter 和 TokenFilter 对具有相关的（即：MappingCharFilter 和 ASCIIFoldingFilter）或几乎相同的（即：PatternReplaceCharFilterFactory 和PatternReplaceFilterFactory）功能，并且它可能并不总是显而易见的，这是最好的选择。  
  
决定使用哪一个 Tokenizer，以及是否需要预处理字符流。  
例如，假设你有一个 tokenizer，如 StandardTokenizer，虽然你对它的工作方式感到满意，但是你想要定制一些特定字符的行为。您可以修改规则并使用 JFlex 重新构建自己的 tokenizer，但是在使用标记和 CharFilter 之前简单地映射某些字符可能更容易。  
