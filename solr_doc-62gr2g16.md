## Solr使用外部文件和进程 
<div class="content-intro view-box "><h2><span style="font-family: inherit; font-size: 16px; font-weight: 600;">ExternalFileField 类型</span>
    </h2>
## <a href="http://lucene.apache.org/solr/guide/7_0/working-with-external-files-and-processes.html#the-externalfilefield-type"/>

ExternalFileField 类型可以指定 Solr 索引之外的文件中字段的值。对于这样的字段，该文件包含从键字段到字段值的映射。另一种方法是，如果在文档中索引时不指定字段，Solr 将在外部文件中查找此字段的值。
      
  
```
Tip：外部字段不可搜索。它们只能用于函数查询或显示。有关函数查询的详细信息, 请参阅函数查询一节。
```

对于您希望在许多文档中更新特定字段的情况，ExternalFileField 类型非常方便，它比您要更新其余文档的时间更频繁。例如，假设您已经根据视图的数量实现了文档等级。您可能需要每天或每小时更新所有文档的等级，而文档的其余内容可能会更少更新。如果没有 ExternalFileField，则需要更新每个文档才能更改排名。使用ExternalFileField 效率更高，因为特定字段的所有文档值都存储在外部文件中，可以根据需要随时更新。
      
  
在 schema.xml 中，这个字段类型的定义可能如下所示：  
```
&lt;fieldType name="entryRankFile" keyField="pkId" defVal="0" stored="false" indexed="false" class="solr.ExternalFileField"/&gt;
```
keyField 属性定义了将在外部文件中定义的键。它通常是索引的唯一键，但只要 keyField 可以用来识别索引中的文档就不需要。defVal 定义了一个默认值，如果外部文件中没有特定文档的条目，将使用该默认值。  

### 外部文件的格式<a href="http://lucene.apache.org/solr/guide/7_0/working-with-external-files-and-processes.html#format-of-the-external-file"/>

该文件本身位于 Solr 的索引目录中，默认情况下是 $SOLR_HOME/data。该文件的名称应该是 external_fieldname_ 或 external_fieldname_.*。对于上面的例子，则文件可以被命名为 external_entryRankFile 或 external_entryRankFile.txt。  
```
Tip：如果出现使用名称模式.*（如.txt）的任何文件，则将使用最后一个 (按名称排序后)，并且将删除以前的版本。此行为支持在可能无法覆盖文件的系统上(例如，在 Windows 上，如果该文件正在使用中)的实现。
```

该文件包含将等号左边的关键字段映射到右边的值的条目。以下是一些示例条目：  
```
doc33=1.414
doc34=3.14159
doc40=42
```

此文件中列出的键不需要是唯一的。不需要对该文件进行排序，但如果是的话，Solr 将能够更快地执行查找。  

### 重新加载外部文件<a href="http://lucene.apache.org/solr/guide/7_0/working-with-external-files-and-processes.html#reloading-an-external-file"/>

您可以定义一个事件侦听器来重新加载外部文件，当搜索器重新加载或者当新的搜索器被启动时。有关更多信息，请参阅 “Query-Related Listeners” 一节，但 solrconfig. xml 中的示例定义可能如下所示：  
```
&lt;listener event="newSearcher" class="org.apache.solr.schema.ExternalFileFieldReloader"/&gt;
&lt;listener event="firstSearcher" class="org.apache.solr.schema.ExternalFileFieldReloader"/&gt;
```


## PreAnalyzedField 类型<a href="http://lucene.apache.org/solr/guide/7_0/working-with-external-files-and-processes.html#the-preanalyzedfield-type"/>

PreAnalyzedField 类型提供了一种方法发送到 Solr 序列化的标记流，可选地使用字段的独立存储值，并且存储和索引这些信息，而不需要在 Solr 中应用任何附加的文本处理。如果用户希望提交已经被一些现有的外部文本处理管道处理的字段内容（例如，已经被标记化、注释、词干、同义词插入等等），而使用 Lucene 的TokenStream 提供的所有丰富的属性（每个令牌属性）。
      
  
序列化格式可以使用 PreAnalyzedParser 接口的实现来插入。有两个现成的实现：  

    - JsonPreAnalyzedParser：顾名思义，它解析使用 JSON 来表示字段内容的内容。如果没有对字段类型进行其他配置，则这是要使用的默认分析器。
    - SimplePreAnalyzedParser：使用简单严格的纯文本格式，在某些情况下可能比 JSON 更容易创建。

只有一个配置参数 parserImpl。此参数的值应该是实现 PreAnalyzedParser 接口的类的完全限定类名称。这个参数的默认值是：org.apache.solr.schema.JsonPreAnalyzedParser。
      
  
默认情况下，这种类型的字段的查询时间分析器将与 "索引时间分析器" 相同，后者需要序列化的分析文本。您必须将一个查询类型分析器添加到您的 fieldType 中，以便对未预先分析的查询执行分析。在下面的例子中，索引时间分析器需要默认的 JSON 序列化格式，而查询时间分析器将使用：StandardTokenizer / LowerCaseFilter：  
```
&lt;fieldType name="pre_with_query_analyzer" class="solr.PreAnalyzedField"&gt;
  &lt;analyzer type="query"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```


### JsonPreAnalyzedParser<a href="http://lucene.apache.org/solr/guide/7_0/working-with-external-files-and-processes.html#jsonpreanalyzedparser"/>

这是 PreAnalyzedField 类型使用的默认序列化格式。它使用带有以下键的顶级 JSON 映射：  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">键</th>
            <th style="text-align: center;">描述</th>
            <th style="text-align: center;">是否需要</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;"><code>v</code>
                  
            </td>
            <td>
                版本密钥。目前支持的版本是<code>1</code>。  
            </td>
            <td>
                <p style="text-align: center;">需要  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>str</code>
                  
            </td>
            <td>
                存储字段的字符串值。您最多可以使用一个<code>str</code>或<code>bin</code>。  
            </td>
            <td>
                <p style="text-align: center;">可选的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>bin</code>
                  
            </td>
            <td>
                存储的字段的二进制值。二进制值必须是 Base64 编码的。  
            </td>
            <td>
                <p style="text-align: center;">可选的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>tokens</code>
                  
            </td>
            <td>
                序列化的令牌流。这是一个 JSON 列表。  
            </td>
            <td>
                <p style="text-align: center;">可选的  
            </td>
        </tr>
    </tbody>
</table>
任何其他顶级密钥都将被忽略。  

### 令牌流序列化

#### <a href="http://lucene.apache.org/solr/guide/7_0/working-with-external-files-and-processes.html#token-stream-serialization"/>

令牌流表示为 JSON 映射的 JSON 列表。每个令牌的映射由以下键和值组成：  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
                    <col/>
                        <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">键</th>
            <th style="text-align: center;">描述</th>
            <th style="text-align: center;">Lucene属性</th>
            <th style="text-align: center;">值</th>
            <th style="text-align: center;">是否需要</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;"><code>t</code>
                  
            </td>
            <td>
                <p style="text-align: center;">令牌  
            </td>
            <td>
                <p style="text-align: center; ">CharTermAttribute  
            </td>
            <td>
                表示当前令牌的 UTF-8 字符串  
            </td>
            <td>
                <p style="text-align: center;">需要  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>s</code>
                  
            </td>
            <td>
                <p style="text-align: center;">起始偏移量  
            </td>
            <td>
                <p style="text-align: center;">OffsetAttribute  
            </td>
            <td>
                非负整数  
            </td>
            <td>
                <p style="text-align: center;">可选的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>e</code>
                  
            </td>
            <td>
                <p style="text-align: center;">结束偏移量  
            </td>
            <td>
                <p style="text-align: center;">OffsetAttribute  
            </td>
            <td>
                非负整数  
            </td>
            <td>
                <p style="text-align: center;">可选的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>i</code>
                  
            </td>
            <td>
                <p style="text-align: center;">位置增量  
            </td>
            <td>
                <p style="text-align: center;">PositionIncrementAttribute  
            </td>
            <td>
                非负整数 - 默认是 <code>1</code>
                  
            </td>
            <td>
                <p style="text-align: center;">可选的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>p</code>
                  
            </td>
            <td>
                <p style="text-align: center;">有效载荷  
            </td>
            <td>
                <p style="text-align: center;">PayloadAttribute  
            </td>
            <td>
                Base64 编码的有效载荷  
            </td>
            <td>
                <p style="text-align: center;">可选的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>y</code>
                  
            </td>
            <td>
                <p style="text-align: center;">词法类型  
            </td>
            <td>
                <p style="text-align: center;">TypeAttribute  
            </td>
            <td>
                UTF-8 字符串  
            </td>
            <td>
                <p style="text-align: center;">可选的  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>f</code>
                  
            </td>
            <td>
                <p style="text-align: center;">标志  
            </td>
            <td>
                <p style="text-align: center;">的FlagsAttribute  
            </td>
            <td>
                以十六进制格式表示整数值的字符串  
            </td>
            <td>
                <p style="text-align: center;">可选的  
            </td>
        </tr>
    </tbody>
</table>
任何其他的关键是默默的忽略。  

### JsonPreAnalyzedParser 示例

```
{
  "v":"1",
  "str":"test ąćęłńóśźż",
  "tokens": [
    {"t":"one","s":123,"e":128,"i":22,"p":"DQ4KDQsODg8=","y":"word"},
    {"t":"two","s":5,"e":8,"i":1,"y":"word"},
    {"t":"three","s":20,"e":22,"i":1,"y":"foobar"}
  ]
}
```


### SimplePreAnalyzedParser<a href="http://lucene.apache.org/solr/guide/7_0/working-with-external-files-and-processes.html#simplepreanalyzedparser"/>

通过 parserImpl 配置参数指定此格式时使用的完全限定类名为 org.apache.solr.schema.SimplePreAnalyzedParser。  

### SimplePreAnalyzedParser 语法

#### <a href="http://lucene.apache.org/solr/guide/7_0/working-with-external-files-and-processes.html#simplepreanalyzedparser-syntax"/>

这个解析器支持的序列化格式如下：  
序列化格式：  
```
content ::= version (stored)? tokens
version ::= digit+ " "
; stored field value - any "=" inside must be escaped!
stored ::= "=" text "="
tokens ::= (token ((" ") + token)*)*
token ::= text ("," attrib)*
attrib ::= name '=' value
name ::= text
value ::= text
```

“文本”值中的特殊字符可以使用转义字符 '\' 进行转义。以下转义序列被识别：
      
  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">EscapeSequence</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;"><code>\</code>
                  
            </td>
            <td>
                <p style="text-align: center;">文字空间字符
                      
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>\,</code>
                  
            </td>
            <td>
                <p style="text-align: center;">文字<code>,</code>字符  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>\=</code>
                  
            </td>
            <td>
                <p style="text-align: center;">文字<code>=</code>字符  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>\\</code>
                  
            </td>
            <td>
                <p style="text-align: center;">文字<code>\</code>字符  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>\n</code>
                  
            </td>
            <td>
                <p style="text-align: center;">新行  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>\r</code>
                  
            </td>
            <td>
                <p style="text-align: center;">回车  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>\t</code>
                  
            </td>
            <td>
                <p style="text-align: center;">水平选项卡  
            </td>
        </tr>
    </tbody>
</table>
请注意，不支持 Unicode 序列（例如，\u0001）。  

### 支持的属性

#### <a href="http://lucene.apache.org/solr/guide/7_0/working-with-external-files-and-processes.html#supported-attributes"/>

以下标记属性受支持，并用短符号名称标识：  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
                    <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">名称</th>
            <th style="text-align: center;">描述</th>
            <th style="text-align: center;">Lucene 属性</th>
            <th style="text-align: center;">值格式</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;"><code>i</code>
                  
            </td>
            <td>
                <p style="text-align: center;">位置增量  
            </td>
            <td>
                <p style="text-align: center; ">PositionIncrementAttribute  
            </td>
            <td>
                <p style="text-align: center;">整数  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>s</code>
                  
            </td>
            <td>
                <p style="text-align: center;">起始偏移量  
            </td>
            <td>
                <p style="text-align: center;">OffsetAttribute  
            </td>
            <td>
                <p style="text-align: center;">整数  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>e</code>
                  
            </td>
            <td>
                <p style="text-align: center;">结束偏移量  
            </td>
            <td>
                <p style="text-align: center;">OffsetAttribute  
            </td>
            <td>
                <p style="text-align: center;">整数  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>y</code>
                  
            </td>
            <td>
                <p style="text-align: center;">词法类型  
            </td>
            <td>
                <p style="text-align: center;">TypeAttribute  
            </td>
            <td>
                <p style="text-align: center;">字符串  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center; "><code>f</code>
                  
            </td>
            <td>
                <p style="text-align: center;">标志  
            </td>
            <td>
                <p style="text-align: center;">FlagsAttribute  
            </td>
            <td>
                <p style="text-align: center;">十六进制整数  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"><code>p</code>
                  
            </td>
            <td>
                <p style="text-align: center;">有效载荷  
            </td>
            <td>
                <p style="text-align: center;">PayloadAttribute  
            </td>
            <td>
                <p style="text-align: center;">十六进制格式的字节; 空白被忽略  
            </td>
        </tr>
    </tbody>
</table>
令牌位置被跟踪并隐式添加到令牌流中 - 开始和结束偏移量仅考虑文本和空白字段，并排除令牌属性占用的空间。
      
  

### 令牌流示例

```
1 one two three
```

    - 版本：1
    - 存储：null
    - 令牌：（term = one，startOffset = 0，endOffset = 3）
    - 标记：（term = two，startOffset = 4，endOffset = 7）
    - 标记：（term = three，startOffset = 8，endOffset = 13）    

```
1 one  two    three
```

    - 版本：1
    - 存储：null
    - 令牌：（term = one，startOffset = 0，endOffset = 3）
    - 标记：（term = two，startOffset = 5，endOffset = 8）
    - 令牌：（term = three，startOffset = 11，endOffset = 16）  

```
1 one,s=123,e=128,i=22 two three,s=20,e=22
```
  
    - 版本：1
    - 存储：null
    - 标记：（term = one，positionIncrement = 22，startOffset = 123，endOffset = 128）
    - 令牌：（term = two，positionIncrement = 1，startOffset = 5，endOffset = 8）
    - 令牌：（term = three，positionIncrement = 1，startOffset = 20，endOffset = 22）   

```
1 \ one\ \,,i=22,a=\, two\=
\n,\ =\ \
```
 
    - 版本：1
    - 存储：null
    - 标记：（term = one ,，positionIncrement = 22，startOffset = 0，endOffset = 6）
    - 令牌：（term = two=，positionIncrement = 1，startOffset = 7，endOffset = 15）
    - 令牌：（term = \，positionIncrement = 1，startOffset = 17，endOffset = 18）

请注意，未知属性及其值将被忽略，因此在本例中，第一个标记上的 “a” 属性和第二个标记上的“”（转义空间）属性与它们的值一起被忽略，因为它们不在受支持的属性名称中。  
```
1 ,i=22 ,i=33,s=2,e=20 ,
```

    - 版本：1
    - 存储：null
    - 令牌：（term =，positionIncrement = 22，startOffset = 0，endOffset = 0）
    - 标记：（term =，positionIncrement = 33，startOffset = 2，endOffset = 20）
    - 标记：（term =，positionIncrement = 1，startOffset = 2，endOffset = 2）  

```
1 =This is the stored part with \=
\n \t escapes.=one two three
```
  
    - 版本：1
    - 存储： This is the stored part with = \t escapes.
    - 令牌：（term = one，startOffset = 0，endOffset = 3）
    - 标记：（term = two，startOffset = 4，endOffset = 7）
    - 标记：（term = three，startOffset = 8，endOffset = 13）

请注意，\t 在上面存储的值不是文字；它显示了以可视方式指示存储值中的实际制表符字符的方法。  
```
1 ==
```

    - 版本：1
    - 存储：“”
    - （没有令牌）  

```
1 =this is a test.=
```
  
    - 版本：1
    - 存储： this is a test.
    - （没有令牌）
