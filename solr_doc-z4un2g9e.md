## Solr中有哪些Tokenizers 
<div class="content-intro view-box ">Tokenizer 负责将字段数据分解为词法单位或标记。  
  
本节将介绍以下的 Tokenizer：  

    - 标准 Tokenizer
    - 经典 Tokenizer
    - 关键字 Tokenizer
    - 信令 Tokenizer
    - 小写 Tokenizer
    - N-gram Tokenizer
    - 边缘 N-gram Tokenizer
    - ICU Tokenizer
    - 路径层次 Tokenizer
    - 正则表达式模式 Tokenizer
    - 简化的正则表达式模式 Tokenizer
    - 简化的正则表达式模式分割 Tokenizer
    - UAX29 URL 电子邮件 Tokenizer
    - 白空间 Tokenizer

您可以在 schema.xml 中使用 &lt;tokenizer&gt; 元素（作为 &lt;analyzer&gt; 的子级）将文本字段类型的 tokenizer 配置：  
```
&lt;fieldType name="text" class="solr.TextField"&gt;
  &lt;analyzer type="index"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.StandardFilterFactory"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```

类属性命名一个工厂类，它将在需要时实例化一个 tokenizer 对象。Tokenizer 工厂类实现 org.apache.solr.analysis.TokenizerFactory。TokenizerFactory 的create() 方法接受一个 Reader 并返回一个 TokenStream。当 Solr 创建 tokenizer 时，它传递一个 Reader 对象来提供文本字段的内容。  
  
通过设置 &lt;tokenizer&gt; 元素的属性，可以将参数传递给 tokenizer 工厂。  
```
&lt;fieldType name="semicolonDelimited" class="solr.TextField"&gt;
  &lt;analyzer type="query"&gt;
    &lt;tokenizer class="solr.PatternTokenizerFactory" pattern="; "/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```

以下各节介绍此版本 Solr 中包含的 tokenizer 工厂类。  
有关 Solr 的 tokenizers 的用户提示，请参阅：http://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters。  

## 标准 Tokenizer

该 tokenizer 将文本字段分割为标记，将空格和标点符号作为分隔符。分隔符字符被丢弃，但以下情况除外：  
  

    - 不后跟空格的句点 (点) 作为标记的一部分保留，包括因特网域名。
    - “@”字符是标记拆分标点符号集合，因此电子邮件地址不会保留为单个标记。

请注意，单词是以连字符分隔的。  
标准分词器支持 Unicode 标准附件 UAX＃29 字边界，并且使用下列标记类型：&lt;ALPHANUM&gt;、&lt;NUM&gt;、&lt;SOUTHEAST_ASIAN&gt;、&lt;IDEOGRAPHIC&gt; 和&lt;HIRAGANA&gt;。  
工厂类： solr.StandardTokenizerFactory  
参数：  
maxTokenLength：（整数，默认值为255）Solr 忽略超过由 maxTokenLength 指定的字符数的标记。  
例如：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
&lt;/analyzer&gt;
```

输入： "Please, email john.doe@foo.com by 03-09, re: m37-xq."  
输出： "Please", "email", "john.doe", "foo.com", "by", "03", "09", "re", "m37", "xq"  

## 经典 Tokenizer

经典 Tokenizer 保留与 Solr 版本3.1和之前的标准 Tokenizer 相同的行为。它不使用标准 Tokenizer 使用的 Unicode 标准附录 UAX＃29 字边界规则。该 tokenizer将文本字段分割为标记，将空格和标点符号作为分隔符。分隔符字符被丢弃，但以下情况除外：  
  

    - 没有被空白的时间段（点）被保存为令牌的一部分。
    - 除非在单词中有一个数字，否则单词将被拆分为连字符，在这种情况下，令牌不会被拆分，数字和连字符将被保留。
    - 识别 Internet 域名和电子邮件地址, 并将其保留为单个标记。  

工厂类： solr.ClassicTokenizerFactory  
参数：  
maxTokenLength：（整数，默认值为 255）Solr 忽略超过由 maxTokenLength 指定的字符数的标记。  
例如：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.ClassicTokenizerFactory"/&gt;
&lt;/analyzer&gt;
```

输出："Please, email john.doe@foo.com by 03-09, re: m37-xq."  
输入："Please", "email", "john.doe@foo.com", "by", "03-09", "re", "m37-xq"  

## 关键字 Tokenizer

这个 tokenizer 将整个文本字段视为单个标记。  
工厂类： solr.KeywordTokenizerFactory  
参数：无  
示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.KeywordTokenizerFactory"/&gt;
&lt;/analyzer&gt;
```

输入："Please, email john.doe@foo.com by 03-09, re: m37-xq."  
输出： "Please, email john.doe@foo.com by 03-09, re: m37-xq."  

## 信令 Tokenizer

该 tokenizer 从连续字母串创建标记，丢弃所有非字母字符。  
工厂类： solr.LetterTokenizerFactory  
参数：无  
示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.LetterTokenizerFactory"/&gt;
&lt;/analyzer&gt;
```

输入： "I can’t."  
输出："I", "can", "t"  

## 小写 Tokenizer

通过以非字母分隔的方式标记输入流，然后将所有字母转换为小写。空白和非字母被丢弃。  
  
工厂类： solr.LowerCaseTokenizerFactory  
参数：无  
示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.LowerCaseTokenizerFactory"/&gt;
&lt;/analyzer&gt;
```

输入： "I just *LOVE* my iPhone!"  
输出： "i", "just", "love", "my", "iphone"  

## N-gram Tokenizer

读取字段文本并在给定范围内生成大小为 n-gram 的记号。  
工厂类： solr.NGramTokenizerFactory  
参数：  
minGramSize：（整数，默认1）最小 n-gram 大小，必须&gt; 0。  
maxGramSize：（整数，默认2）最大 n-gram 大小，必须&gt; = minGramSize。  
示例：  
默认行为。请注意，这个 tokenizer 操作整个领域。它不会在空格处破坏字段。因此，空格字符被包括在编码中。  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.NGramTokenizerFactory"/&gt;
&lt;/analyzer&gt;
```

输入："hey man"  
输出："h", "e", "y", " ", "m", "a", "n", "he", "ey", "y ", " m", "ma", "an"  
示例：  
n-gram 大小范围为4到5：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.NGramTokenizerFactory" minGramSize="4" maxGramSize="5"/&gt;
&lt;/analyzer&gt;
```

输入："bicycle"  
输出："bicy", "bicyc", "icyc", "icycl", "cycl", "cycle", "ycle"  

## 边缘 N-gram Tokenizer

读取字段文本并在给定范围内生成大小的边缘 n-gram 标记。  
  
工厂类： solr.EdgeNGramTokenizerFactory  
参数：  
minGramSize：（整数，默认值为1）最小 n-gram 大小，必须&gt; 0。  
maxGramSize：（整数，默认值为1）最大 n-gram 大小，必须&gt; = minGramSize。  
side：（"front" 或 "back"，默认为"front"）是否从文本的开头 (front) 或从末尾 (back) 计算 n-gram。  
示例：  
默认行为（最小值和最大值默认为1）：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.EdgeNGramTokenizerFactory"/&gt;
&lt;/analyzer&gt;
```

输入： “babaloo”  
输出： “b”  
示例：  
边缘 n-gram 范围为2到5  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.EdgeNGramTokenizerFactory" minGramSize="2" maxGramSize="5"/&gt;
&lt;/analyzer&gt;
```

输入： “babaloo”  
输出： “ba”，“bab”，“baba”，“babal”  
示例：  
边缘 n-gram 范围从2到5，从背面：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.EdgeNGramTokenizerFactory" minGramSize="2" maxGramSize="5" side="back"/&gt;
&lt;/analyzer&gt;
```

输入： “babaloo”  
输出： “oo”，“loo”，“aloo”，“baloo”  

## ICU Tokenizer

这个 tokenizer 处理多语言文本，并根据其脚本属性对其进行标记。  
  
您可以通过指定每个脚本规则文件来自定义此标记器的行为。要添加每个脚本规则，请添加一个 rulefiles 参数，该参数应包含以下格式的 code:rulefile 对的逗号分隔列表对象：四个字母的 ISO 15924 脚本代码，后跟一个冒号，然后是一个资源路径。例如，要指定 Latin（脚本代码 “Latn”）和 Cyrillic（脚本代码 “Cyrl”）的规则，您可以输入：Latn:my.Latin.rules.rbbi,Cyrl:my.Cyrillic.rules.rbbi。  
solr.ICUTokenizerFactory 的默认配置提供了 UAX＃29 的分词规则标记（如 solr.StandardTokenizer），但也包括自定义定制为 Hebrew（专门处理双引号和单引号），为 Khmer，Lao 和 Myanmar 的音节标记，并对CJK字符进行基于字典的分词。  
工厂类： solr.ICUTokenizerFactory  
参数：  
rulefile：以下格式的 code:rulefile 对的逗号分隔列表对象：四个字母的 ISO 15924 脚本代码，后跟一个冒号，然后是一个资源路径。  
示例：  
```
&lt;analyzer&gt;
  &lt;!-- no customization --&gt;
  &lt;tokenizer class="solr.ICUTokenizerFactory"/&gt;
&lt;/analyzer&gt;
```
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.ICUTokenizerFactory"
             rulefiles="Latn:my.Latin.rules.rbbi,Cyrl:my.Cyrillic.rules.rbbi"/&gt;
&lt;/analyzer&gt;
```
要使用此 tokenizer，您必须将其他 . jar 添加到 Solr 的类路径中 (如 SolrConfig 中的 Lib 指令中所述)。有关需要添加到您的 SOLR_HOME/lib 的 jar 的信息，请参见 solr/contrib/analysis-extras/README.txt。  

## 路径层次 Tokenizer

这个 tokenizer 从文件路径层次结构中创建同义词。  
  
工厂类： solr.PathHierarchyTokenizerFactory  
参数：  
delimiter：（字符，无默认值）您可以指定文件路径分隔符并将其替换为您提供的分隔符。这对于使用反斜线分隔符是非常有用的。  
replace：（字符，无默认值）指定 Solr 在标记化输出中使用的分隔符。  
示例：  
```
&lt;fieldType name="text_path" class="solr.TextField" positionIncrementGap="100"&gt;
  &lt;analyzer&gt;
    &lt;tokenizer class="solr.PathHierarchyTokenizerFactory" delimiter="\" replace="/"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```

输入： “c：\ usr \ local \ apache”  
输出： “c：”，“c：/ usr”，“c：/ usr / local”，“c：/ usr / local / apache”  

## 正则表达式模式 Tokenizer

该 tokenizer 使用 Java 正则表达式将输入文本流分解为标记。由 pattern 参数提供的表达式可以被解释为分隔符，或者匹配应该从文本中提取的作为记号的模式。  
有关 Java 正则表达式语法的更多信息，请参阅 Javadocsjava.util.regex.Pattern。  
工厂类： solr.PatternTokenizerFactory  
参数：    
pattern：（必需的）正则表达式，如 java.util.regex.Pattern 中所定义。  
  
group：（可选，默认为-1）指定要将哪个正则表达式组提取为标记。值-1意味着正则表达式应被视为分隔符的分隔符。非负数组（&gt; = 0）表示匹配该正则表达式组的字符序列应转换为令牌。组0指整个正则表达式，大于零的组引用正则表达式的带括号的子表达式，从左到右计数。  
示例：  
用逗号分隔的列表。令牌由一系列零个或多个空格，一个逗号和零个或多个空格分隔。  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.PatternTokenizerFactory" pattern="\s*,\s*"/&gt;
&lt;/analyzer&gt;
```

输入："fee,fie, foe , fum, foo"  
输出："fee", "fie", "foe", "fum", "foo"  
示例：  
提取简单的大写单词。至少一个大写字母后跟零个或多个字母的序列被提取为一个标记。  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.PatternTokenizerFactory" pattern="[A-Z][A-Za-z]*" group="0"/&gt;
&lt;/analyzer&gt;
```

输入： "Hello. My name is Inigo Montoya. You killed my father. Prepare to die."  
输出："Hello", "My", "Inigo", "Montoya", "You", "Prepare"  
示例：  
使用可选的分号分隔符提取以 “SKU”、“Part”或 “Part Number” 开头的零件号码，区分大小写。部件号必须是全部数字，并带有可选的连字符。正则表达式捕获组通过从左到右计算左括号进行编号。组3是子表达式“[0-9 - ] +”，它匹配一个或多个数字或连字符。  
  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.PatternTokenizerFactory" pattern="(SKU|Part(\sNumber)?):?\s(\[0-9-\]+)" group="3"/&gt;
&lt;/analyzer&gt;
```

输入： "SKU: 1234, Part Number 5678, Part: 126-987"  
输出： "1234", "5678", "126-987"  

## 简化的正则表达式模式 Tokenizer

这个 tokenizer 类似于上面描述的 PatternTokenizerFactory，但是使用 Lucene RegExp 模式匹配为输入流构造不同的标记。语法比 PatternTokenizerFactory 限制更多，但是标记化速度要快得多。  
工厂类： solr.SimplePatternTokenizerFactory  
参数：  
pattern：（必需）在 RegExpjavadoc 中定义的正则表达式，标识要包含在标记中的字符。这个匹配是贪婪的，这样就可以创建一个给定点上最长的令牌匹配。空令牌永远不会被创建。  
maxDeterminizedStates：（可选，默认为10000）由 regexp 计算的确定自动机的总状态计数限制。  
示例：  
匹配由简单的空格字符分隔的令牌：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.SimplePatternTokenizerFactory" pattern="[^ \t\r\n]+"/&gt;
&lt;/analyzer&gt;
```


## 简化的正则表达式模式分割 Tokenizer

这个 tokenizer 类似于上面描述的 SimplePatternTokenizerFactory，但是使用 Lucene RegExp 模式匹配来标识应该用于分割标记的字符序列。语法比 PatternTokenizerFactory 限制更多，但是标记化速度要快得多。  
工厂类： solr.SimplePatternSplitTokenizerFactory  
参数：  
pattern：（必需）在 RegExpjavadocs 中定义的正则表达式，标识应该分割记号的字符。该匹配是贪婪的，使得在给定点匹配最长的令牌分隔符匹配。空令牌永远不会被创建。  
maxDeterminizedStates：（可选，默认为10000）由 regexp 计算的确定自动机的总状态计数限制。  
示例：  
匹配由简单的空格字符分隔的令牌：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.SimplePatternSplitTokenizerFactory" pattern="[ \t\r\n]+"/&gt;
&lt;/analyzer&gt;
```


## UAX29 URL 电子邮件 Tokenizer

该 tokenizer 将文本字段分割为标记，将空格和标点符号作为分隔符。分隔符字符被丢弃，但以下情况除外：  

    - 未后跟空格的句点 (点) 作为标记的一部分保留。
    - 除非在单词中有一个数字，否则单词将被拆分为连字符，在这种情况下，令牌不会被拆分，数字和连字符将被保留。
    - 识别并保留为单个标记如下：包含顶级域名的互联网域名在生成分词器时根据 IANA 根区数据库中的白名单进行验证电子邮件地址：file://、http(s):// 和 ftp:// URLs，IPv4 和 IPv6 地址

该 UAX29 URL 的电子邮件标记生成器支持 Unicode 标准附件 UAX#29 字边界与以下标记类型：&lt;ALPHANUM&gt;、&lt;NUM&gt;、&lt;URL&gt;、&lt;EMAIL&gt;、&lt;SOUTHEAST_ASIAN&gt;、&lt;IDEOGRAPHIC&gt; 和 &lt;HIRAGANA&gt;。  
工厂类： solr.UAX29URLEmailTokenizerFactory  
参数：  
maxTokenLength：（整数，默认值为255）Solr 忽略超过由 maxTokenLength 指定的字符数的标记。  
示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.UAX29URLEmailTokenizerFactory"/&gt;
&lt;/analyzer&gt;
```

输入："Visit http://accarol.com/contact.htm?from=external&amp;a=10 or e-mail bob.cratchet@accarol.com"  
输出： "Visit", "http://accarol.com/contact.htm?from=external&amp;a=10", "or", "e", "mail", "bob.cratchet@accarol.com"  

## 白空间 Tokenizer

简单的 tokenizer，将文本流分割成空白字符，并返回非空白字符序列作为标记。请注意，标记中将包含任何标点符号。  
工厂类： solr.WhitespaceTokenizerFactory  
参数：： rule 指定如何为标记化目的定义空白。有效值如下：  

    - java：（默认）使用 Character.isWhitespace（int）
    - unicode：使用 Unicode 的 WHITESPACE 属性

示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.WhitespaceTokenizerFactory" rule="java" /&gt;
&lt;/analyzer&gt;
```

输入："To be, or what?"  
输出："To", "be,", "or", "what?"  
