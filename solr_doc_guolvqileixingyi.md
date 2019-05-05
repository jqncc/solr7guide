## Solr过滤器类型（一） 
<div class="content-intro view-box ">Solr 过滤器可以检查一个令牌流，并保留它们，根据所使用的过滤器类型对它们进行转换或丢弃。  
  
您可以使用在 schema.xml 中的 &lt;filter&gt; 元素作为子元素来配置每个过滤器，在 &lt;tokenizer&gt; 元素之后。过滤器定义应遵循 tokenizer 或其他过滤器定义，因为它们采用 TokenStream 作为输入。例如：  
```
&lt;fieldType name="text" class="solr.TextField"&gt;
  &lt;analyzer type="index"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;...
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
类属性命名一个工厂类，它将根据需要实例化一个过滤器对象。过滤器工厂类必须实现 org.apache.solr.analysis.TokenFilterFactory 接口。像 tokenizers 一样，过滤器也是 TokenStream 的实例，因此是令牌的生产者。与 tokenizers 不同，过滤器也消耗 TokenStream 中的标记。这允许您按照您喜欢的任何顺序在 tokenizers 下游混合和匹配过滤器。  
  
通过在 &lt;filter&gt; 元素上设置属性，可以将参数传递给 tokenizer 工厂以修改其行为。例如：  
```
&lt;fieldType name="semicolonDelimited" class="solr.TextField"&gt;
  &lt;analyzer type="query"&gt;
    &lt;tokenizer class="solr.PatternTokenizerFactory" pattern="; " /&gt;
    &lt;filter class="solr.LengthFilterFactory" min="2" max="7"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```
以下各节介绍了此版本 Solr 中包含的过滤器工厂。  
有关 Solr 过滤器的用户提示，请参阅：http://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters。  
## ASCII 折叠过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#ascii-folding-filter"/>
此筛选器将不在 Basic Latin Unicode 块（前 127 个 ASCII 字符）中的字母、数字和符号 Unicode 字符转换为其 ASCII 等效字符（如果存在）。此过滤器转换来自以下 Unicode 块的字符：   
  
- [C1控件和拉丁语-1补充](http://www.unicode.org/charts/PDF/U0080.pdf)（PDF）
- [拉丁文扩展-A](http://www.unicode.org/charts/PDF/U0100.pdf)（PDF）
- [拉丁文扩展-B](http://www.unicode.org/charts/PDF/U0180.pdf)（PDF）
- [拉丁文扩展附加](http://www.unicode.org/charts/PDF/U1E00.pdf)（PDF）
- [拉丁文扩展-C](http://www.unicode.org/charts/PDF/U2C60.pdf)（PDF）
- [拉丁文扩展-D](http://www.unicode.org/charts/PDF/UA720.pdf)（PDF）
- [IPA扩展](http://www.unicode.org/charts/PDF/U0250.pdf)（PDF）
- [语音扩展](http://www.unicode.org/charts/PDF/U1D00.pdf)（PDF）
- [语音扩展补充](http://www.unicode.org/charts/PDF/U1D80.pdf)（PDF）
- [一般标点符号](http://www.unicode.org/charts/PDF/U2000.pdf)（PDF）
- [上标和下标](http://www.unicode.org/charts/PDF/U2070.pdf)（PDF）
- [封闭的字母数字](http://www.unicode.org/charts/PDF/U2460.pdf)（PDF）
- [装饰](http://www.unicode.org/charts/PDF/U2700.pdf)（PDF）
- [补充标点符号](http://www.unicode.org/charts/PDF/U2E00.pdf)（PDF）
- [按字母顺序排列的表格](http://www.unicode.org/charts/PDF/UFB00.pdf)（PDF）
- [半宽和全宽窗体](http://www.unicode.org/charts/PDF/UFF00.pdf)（PDF）
工厂类： solr.ASCIIFoldingFilterFactory  
参数：  
**preserveOriginal**
（boolean，默认为false）如果为true，则保留原始标记：“thé” - &gt;“the”，“thé”  

示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.WhitespaceTokenizer"/&gt;
  &lt;filter class="solr.ASCIIFoldingFilterFactory" preserveOriginal="false" /&gt;
&lt;/analyzer&gt;
```
输入： “á”（Unicode 字符 00E1）  
输出： “a”（ASCII 字符 97）  
## Beider-Morse 过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#beider-morse-filter"/>
实现了 Beider-Morse Phonetic Matching (BMPM) 算法，该算法允许识别相似的名称，即使它们拼写不同或使用不同的语言。有关如何工作的更多信息，请参见拼音匹配部分。  
```
Tip：BeiderMorseFilter 由于对 BMPM 算法 3.04 版本的更新而改变了其在 Solr 5.0 中的行为。旧版本的 Solr 实现了 BMPM 版本 3.00 (参见 http://stevemorse.org/phoneticinfo.htm)。使用此过滤器与早期版本的 Solr 建立的任何索引都需要重建。
```
工厂类： solr.BeiderMorseFilterFactory  
参数：  
  
**nameType**
名字的类型。有效值是 GENERIC、ASHKENAZI 或 SEPHARDIC。如果不是处理 Ashkenazi 或 Sephardic 名字，请使用 GENERIC。  
**ruleType**
适用的规则类型。有效值是 APPROX 或 EXACT。  
**concat**
定义多个可能的匹配是否应该与管道（“|”）组合。  
**languageSet**
设置使用的语言。值 “auto” 将允许过滤器识别语言，或者可以提供逗号分隔列表。  

示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.BeiderMorseFilterFactory" nameType="GENERIC" ruleType="APPROX" concat="true" languageSet="auto"&gt;
  &lt;/filter&gt;
&lt;/analyzer&gt;
```
## 经典过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#classic-filter"/>
该过滤器使用经典 Tokenizer 的输出，并从所有格中去除首字母缩写词和 “s” 的句号。  
工厂类： solr.ClassicFilterFactory  
参数：无  
示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.ClassicTokenizerFactory"/&gt;
  &lt;filter class="solr.ClassicFilterFactory"/&gt;
&lt;/analyzer&gt;
```
输入："I.B.M. cat’s can’t"  
Tokenizer 过滤： "I.B.M", "cat’s", "can’t"  
输出："IBM", "cat", "can’t"  
## 普通 Grams 过滤器

## <a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#common-grams-filter"/>
此过滤器通过将常用标记（如 "停止单词" 和 "常规标记"）组合在一起来创建词汇组合。这对创建包含常用词的短语查询非常有用，如 “cat”。Solr 通常会忽略在查询短语中的停止单词，因此搜索 “the cat” 会返回单词 “cat” 的所有匹配。  
  
工厂类： solr.CommonGramsFilterFactory  
参数：  
**words**
（.txt 格式的常用单词文件）提供常用单词文件的名称，例如<code>stopwords.txt</code>。  
**format**
（可选）如果停用词表已经为 Snowball 格式化，您可以指定<code>format="snowball"</code>Solr 可以读取停用词文件。  
**ignoreCase**
（boolean 类型值）如果为 true，则在将它们与常用单词文件进行比较时，该过滤器将忽略单词的大小写。默认值是false。  

示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.CommonGramsFilterFactory" words="stopwords.txt" ignoreCase="true"/&gt;
&lt;/analyzer&gt;
```
输入："the Cat"  
Tokenizer 过滤： "the", "Cat"  
输出： “the_cat”  
## 排序键过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#collation-key-filter"/>
整理允许以对语言敏感的方式排序文本。它通常用于排序，但也可以用于高级搜索。我们在 Unicode 排序部分详细介绍了这一点。  
  
## Daitch-Mokotoff Soundex 过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#daitch-mokotoff-soundex-filter"/>
实现了 Daitch-Mokotoff Soundex 算法，该算法允许识别相似的名称，即使它们拼写有所不同。有关如何工作的更多信息，请参见拼音匹配部分。  
  
工厂类： solr.DaitchMokotoffSoundexFilterFactory  
参数：  
**inject**
（true / false）如果为 true（默认），则新的语音标记被添加到流中。否则，令牌替换为语音等价物。将其设置为 false 将启用拼音匹配，但目标单词的确切拼写可能不匹配。  

示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.DaitchMokotoffSoundexFilterFactory" inject="true"/&gt;
&lt;/analyzer&gt;
```
## 双 Metaphone 过滤器

## <a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#double-metaphone-filter"/>
这个过滤器使用 DoubleMetaphone 编码算法从共同性-编解码器创建令牌。有关更多信息，请参阅拼音部分。  
  
工厂类： solr.DoubleMetaphoneFilterFactory  
参数：  
**inject**
（true / false）如果为true（默认），则新的语音标记被添加到流中。否则，令牌替换为语音等价物。将其设置为 false 将启用拼音匹配，但目标单词的确切拼写可能不匹配。  
**maxCodeLength**
（整数）要生成的代码的最大长度。  

示例：  
注入（true）的默认行为：保留原始标记并将语音标记添加到同一位置。  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.DoubleMetaphoneFilterFactory"/&gt;
&lt;/analyzer&gt;
```
输入： "four score and Kuczewski"  
Tokenizer 过滤："four"(1), "score"(2), "and"(3), "Kuczewski"(4)  
输出： "four"(1), "FR"(1), "score"(2), "SKR"(2), "and"(3), "ANT"(3), "Kuczewski"(4), "KSSK"(4), "KXFS"(4)  
语音标记的位置增量为 0，表示它们与它们从之前导出的标记位于相同的位置。请注意，“Kuczewski” 有两个编码，它们在同一位置添加。  
  
示例：  
放弃原始标记（inject="false"）。  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.DoubleMetaphoneFilterFactory" inject="false"/&gt;
&lt;/analyzer&gt;
```
输入： "four score and Kuczewski"  
Tokenizer 过滤：  "four"(1), "score"(2), "and"(3), "Kuczewski"(4)  
输出："FR"(1), "SKR"(2), "ANT"(3), "KSSK"(4), "KXFS"(4)  
请注意，“Kuczewski” 有两个编码，在相同位置添加。  
## 边缘 N-gram 过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#edge-n-gram-filter"/>
这个过滤器在给定的范围内生成大小的边缘 n-gram 标记。  
  
工厂类： solr.EdgeNGramFilterFactory  
参数：  
**minGramSize**
（整数，默认1）最小 gram 大小。  
**maxGramSize**
（整数，默认1）最大 gram 大小。  

示例：  
默认行为。  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.EdgeNGramFilterFactory"/&gt;
&lt;/analyzer&gt;
```
输入："four score and twenty"  
Tokenizer 过滤："four", "score", "and", "twenty"  
输出："f", "s", "a", "t"  
示例：  
范围从1到4。  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.EdgeNGramFilterFactory" minGramSize="1" maxGramSize="4"/&gt;
&lt;/analyzer&gt;
```
输入： "four score"  
Tokenizer 过滤器："four", "score"  
输出："f", "fo", "fou", "four", "s", "sc", "sco", "scor"  
示例：  
范围从4到6。  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.EdgeNGramFilterFactory" minGramSize="4" maxGramSize="6"/&gt;
&lt;/analyzer&gt;
```
输入："four score and twenty"  
Tokenizer 过滤："four", "score", "and", "twenty"  
输出："four", "scor", "score", "twen", "twent", "twenty"  
## 英语最小阀杆过滤器
这个过滤器把复数的英语单词变成单数形式。  
  
工厂类： solr.EnglishMinimalStemFilterFactory  
参数：无  
示例：  
```
&lt;analyzer type="index"&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.EnglishMinimalStemFilterFactory"/&gt;
&lt;/analyzer&gt;
```
输入： "dogs cats"  
Tokenizer 过滤： "dogs", "cats"  
输出："dog", "cat"  
<h2>英语所有格过滤器  
<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#english-possessive-filter"/></h2>该过滤器从单词中移除奇异的所有格(尾随的 s')。请注意，复数的所有格，例如 “divers' snorkels” 中的“ s' ”，并没有被这个过滤器移除。  
工厂类： solr.EnglishPossessiveFilterFactory  
参数：无  
示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.WhitespaceTokenizerFactory"/&gt;
  &lt;filter class="solr.EnglishPossessiveFilterFactory"/&gt;
&lt;/analyzer&gt;
```
输入： "Man’s dog bites dogs' man"  
Tokenizer 过滤："Man’s", "dog", "bites", "dogs'", "man"  
输出："Man", "dog", "bites", "dogs'", "man"  
## 指纹过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#fingerprint-filter"/>
该过滤器输出单个令牌，该令牌是经过排序和去重的一组输入令牌的串联。这对集群/链接用例很有用。  
  
工厂类： solr.FingerprintFilterFactory  
参数：  
**separator**
用于分隔令牌的字符组合成单个输出令牌。默认为 “”（空格字符）。  
**maxOutputTokenSize**
摘要输出令牌的最大长度。如果超过，则不会输出令牌。默认为1024。  

示例：  
```
&lt;analyzer type="index"&gt;
  &lt;tokenizer class="solr.WhitespaceTokenizerFactory"/&gt;
  &lt;filter class="solr.FingerprintFilterFactory" separator="_" /&gt;
&lt;/analyzer&gt;
```
输入： "the quick brown fox jumped over the lazy dog"  
Tokenizer 过滤器： “the”，“quick”，“brown”，“fox”，“jumped”，“over”，“the”，“lazy”，“dog”  
输出："brown_dog_fox_jumped_lazy_over_quick_the"  
## 平展图形过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#flatten-graph-filter"/>
此过滤器必须包含在索引时间分析器规范中，该规范至少包含一个图形识别过滤器，包括同义词图形过滤器和字符分隔符图形过滤器。  
工厂类： solr.FlattenGraphFilterFactory  
参数：无  
有关同义词图形筛选器和字符分隔符图形筛选器，请参阅下面的示例。  
## Hunspell Stem 过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#hunspell-stem-filter"/>
在 Hunspell Stem Filter 提供几种语言的支持。您必须为您希望用于 Hunspell Stem Filter 的每种语言提供 dictionary（.dic）和 rules（.aff）文件。  
你可以[在这里](http://wiki.services.openoffice.org/wiki/Dictionaries)下载这些语言文件。  
请注意，根据提供的词典和规则文件的质量，结果会有很大差异。例如，一些语言只有一个最小的单词列表，没有形态信息。另一方面，对于没有词干的语言，但有一个广泛的字典文件，Hunspell 词干可能是一个不错的选择。  
工厂类： solr.HunspellStemFilterFactory  
参数：  
**dictionary**
（必填）字典文件的路径。  
**affix**
（必需）规则文件的路径。  
**ignoreCase**
（boolean）控制匹配是否区分大小写。默认值是 false。  
**strictAffixParsing**
（boolean）控制词缀解析是否严格。如果为 true，则读取附加规则时出错将导致 ParseException，否则将被忽略。默认值是 true。  

示例：  
```
&lt;analyzer type="index"&gt;
  &lt;tokenizer class="solr.WhitespaceTokenizerFactory"/&gt;
  &lt;filter class="solr.HunspellStemFilterFactory"
    dictionary="en_GB.dic"
    affix="en_GB.aff"
    ignoreCase="true"
    strictAffixParsing="true" /&gt;
&lt;/analyzer&gt;
```
输入： "jump jumping jumped"  
Tokenizer 过滤器："jump", "jumping", "jumped"  
输出："jump", "jump", "jump"  
## 连字过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#hyphenated-words-filter"/>
这个过滤器重建由于字段测试中的换行符或其他介入的空白字符而被标记为两个标记的连字符。如果令牌以连字符结尾，则使用下面的令牌连接，连字符将被丢弃。  
请注意，为使此过滤器正常工作，上游标记化程序不得删除尾部连字符。这个过滤器通常只在索引时有用。  
工厂类： solr.HyphenatedWordsFilterFactory  
参数：无  
示例：  
```
&lt;analyzer type="index"&gt;
  &lt;tokenizer class="solr.WhitespaceTokenizerFactory"/&gt;
  &lt;filter class="solr.HyphenatedWordsFilterFactory"/&gt;
&lt;/analyzer&gt;
```
输入："A hyphen- ated word"  
Tokenizer 过滤： "A", "hyphen-", "ated", "word"  
输出："A", "hyphenated", "word"  
## ICU 折叠过滤器<a href="http://lucene.apache.org/solr/guide/7_0/filter-descriptions.html#icu-folding-filter"/>
此过滤器是一种自定义 Unicode 规范化表单，它应用可在 unicode 技术报告30中指定的 NFKC_Casefold，以及在 ICU 标准件2过滤器中所描述的标准化格式化形式。此过滤器是 ASCII 折叠过滤器，小写过滤器和 ICU 规范器2过滤器的组合行为的更好替代品。NFKC_Casefold  
  
要使用此过滤器，请参阅 solr/contrib/analysis-extras/README.txt 以获取您需要添加到您的 solr_home/lib 的 jar 的说明。有关添加 jar 的更多信息，请参阅Solrconfig 中的 Lib 指令部分。  
工厂类： solr.ICUFoldingFilterFactory  
参数：无  
示例：  
```
&lt;analyzer&gt;
  &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
  &lt;filter class="solr.ICUFoldingFilterFactory"/&gt;
&lt;/analyzer&gt;
```
有关此规范化表单的详细信息，请参阅：http://www.unicode.org/reports/tr30/tr30-4.html。  
