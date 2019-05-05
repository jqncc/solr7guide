## Solr扩展的DisMax查询解析器：eDismax 
<div class="content-intro view-box ">Solr 扩展的 DisMax（eDisMax）查询解析器是 DisMax 查询解析器的改进版本。
      
  
除了支持所有的 DisMax 查询解析器参数外，eDismax 解析器还有如下扩展：  

    - 支持完整的 Lucene 查询分析器语法，它与 Solr 的标准查询解析器具有相同的增强功能。
        <ul>
            <li>支持 AND、OR、NOT、 - 和+等查询。
            - 在 Lucene 语法模式下可选地将 “and” 和 “or” 视为 “AND” 和 “OR”。
- 尊重 'magic 字段' 的名字：_val_ 和 _query_。这些在 Schema 中不是真实的字段，但是如果使用它，它可以帮助做特殊的事情（如 _val_ 的情况下的函数查询, 或者在 _query_ 中的嵌套查询）。如果 _val_ 用于术语或短语查询，则该值将作为函数进行分析。
    </li>
    <li>在语法错误的情况下包括改进的智能部分转义; 在此模式下仍然支持字段查询、+/- 和短语查询。</li>
    <li>通过使用单词 shingles 来提高接近度增强；在应用近似增强之前，不需要查询来匹配文档中的所有单词。</li>
    <li>包括高级的停用词处理：在查询的强制部分中不需要停用词，但仍用于近似增强部分。如果一个查询由所有的停用词组成，例如“to be or not to be”，那么所有的单词都是必需的。
          
    </li>
    <li>包括改进的 boost 功能：在 eDisMax 中，该 boost 功能是一个乘数而不是加数，提高了您的 boost 效果；DisMax的附加 boost 功能（bf 和 bq）也被支持。</li>
    <li>支持纯粹的消极嵌套查询：诸如 +foo (-foo) 的查询将匹配所有文档。</li>
    <li>让您指定允许最终用户查询哪些字段，并禁止直接派遣的搜索。</li>
</ul>

## eDisMax 参数

除了包含所有的 DisMax 参数外，扩展的 DisMax（eDismax）解析器还包括下述的查询参数：  

**sow**

    
        拆分空格。如果设置为<code>true</code>，则对每个单独的空格分隔的术语分别调用文本分析。默认是<code>false</code>；空格分隔的术语序列将一次性提供给文本分析，从而使分析筛选器的功能能够正常运行，例如，多字同义词和 shingles。  
    
**mm.autoRelax**

    
        如果为 true，从一些（而不是所有）qf 字段中删除子句（例如通过停用词过滤器），则所需的子句数（最小值应匹配）将自动放宽。如果您遇到由于<code>qf</code>字段之间不均匀禁止删除而导致查询返回零击中，请使用此参数作为解决方法。  
        
            请注意，放松<code>mm</code>可能会导致不必要的副作用，例如破坏搜索的精确度，具体取决于索引内容的性质。  
          
    
**boost**

    
        分析为查询的字符串的多值列表，并将其分数乘以所有匹配文档的主查询的分数。此参数是使用 BoostQParserPlugin 来包装 eDisMax 生成的查询的简写。  
    
**lowercaseOperators**

    
        指示是否将小写“and”和“or”当作与运算符“AND”和“OR”相同的布尔型参数。默认为<code>false</code>。  
    
**ps**

    
        短语 Slop。slop 的默认量-术语之间的距离-在使用 pf、pf2 或 pf3 字段(影响 boosting)构建的短语查询中。另请参见下面的 “使用‘Slop’” 一节。  
    
**pf2**

    
        带有可选权重的字段的多值列表。类似于<code>pf</code>，但是基于一对单词 shingles。  
    
**ps2**

    
        这类似于<code>ps</code>但是覆盖了用于<code>pf2</code>的 slop 因子。如果未指定，则使用<code>ps</code>。  
    
**pf3**

    
        一个多值字段列表，带有可选的权重，基于单词组的三联体。类似<code>pf</code>，不同之处在于它不是在每个字段中使用输入中的所有单词构建一个短语，而是它会为每个字段生成单词 “shingles”的三联体。  
    
**ps3**

    
        类似于<code>ps</code>但是覆盖了用于<code>pf3</code>的坡度因子。如果未指定，则使用<code>ps</code>。  
    
**stopwords**

    
        一个布尔参数，指示<code>StopFilterFactory</code>在解析查询时是否应该遵守查询分析器中的配置。如果设置为<code>false</code>，那么<code>StopFilterFactory</code>在查询分析器中被忽略。  
    
**uf**

    
        指定允许最终用户明确查询的架构字段。该参数支持通配符。默认是允许所有的字段，相当于<code>uf=*</code>。只允许标题字段使用<code>uf=title</code>。要允许标题和以'_s' 结尾的所有字段，请使用<code>uf=title,*_s</code>。要允许除标题以外的所有字段，请使用<code>uf=*,-title</code>。要禁止所有派出的搜索，请使用<code>uf=-*</code>。  
    


### 使用 Per-Field qf 覆盖的字段别名

可以指定 qf 参数的 Per-field 覆盖，以提供在查询字符串中指定的字段名的1到多个别名，用于在底层查询中使用的字段名。默认情况下，不使用别名，并将查询字符串中指定的字段名称视为索引中的文字字段名称。  

## eDismax 查询的示例

本节中的所有示例 URL 都假定您正在运行 Solr 的 techproducts 示例：  
```
bin/solr -e techproducts
```

根据文档的流行度提升查询词 “hello” 的结果：  
```
http://localhost:8983/solr/techproducts/select?defType=edismax&amp;q=hello&amp;pf=text&amp;qf=text&amp;boost=popularity
```

搜索 iPod 或视频：  
```
http://localhost:8983/solr/techproducts/select?defType=edismax&amp;q=ipod+OR+video
```

在多个字段中搜索，指定（通过 boosts）每个字段相对于彼此的重要性：  
```
http://localhost:8983/solr/techproducts/select?q=video&amp;defType=edismax&amp;qf=features^20.0+text^0.3
```

您可以提高具有与特定值匹配的字段的结果：  
```
http://localhost:8983/solr/techproducts/select?q=video&amp;defType=edismax&amp;qf=features^20.0+text^0.3&amp;bq=cat:electronics^5.0
```

使用“mm”参数，1和2个单词查询要求所有可选子句匹配，但对于具有三个或更多子句的查询，允许使用一个缺失子句：  
```
http://localhost:8983/solr/techproducts/select?q=belkin+ipod&amp;defType=edismax&amp;mm=2
http://localhost:8983/solr/techproducts/select?q=belkin+ipod+gibberish&amp;defType=edismax&amp;mm=2
http://localhost:8983/solr/techproducts/select?q=belkin+ipod+apple&amp;defType=edismax&amp;mm=2
```

在下面的示例中，我们看到 qf 参数的 per-field 覆盖被用于查询字符串中"name" 的别名：“last_name” 和 “first_name” 字段:  
```
defType=edismax
q=sysadmin name:Mike
qf=title text last_name first_name
f.name.qf=last_name first_name
```


## 使用负面的 Boost

“查询”对象级长时间支持负查询提升（导致匹配文档为负值）。现在 QueryParsers 已经被更新来处理这个。  

## 使用 “Slop”

Dismax 和 Edismax 可以针对所有查询字段运行查询，还可以针对短语字段以短语的形式运行查询。但是，该短语查询可能会有一个“slop”，这是查询词语之间的距离，同时仍然将其视为词组匹配。例如：  
```
q=foo bar
qf=field1^5 field2^10
pf=field1^50 field2^20
defType=dismax
```

使用这些参数，Dismax 查询解析器生成一个如下所示的查询：  
```
 (+(field1:foo^5 OR field2:foo^10) AND (field1:bar^5 OR field2:bar^10))
```

但是它也会生成另一个只用于提高结果的查询：  
```
field1:"foo bar"^50 OR field2:"foo bar"^20
```

因此，任何有“foo”和“bar”这个词的文档都会匹配；然而，如果其中一些文件中有两个词作为短语，那么它将得分高得多，因为它更相关。  
如果添加参数 ps（短语 slop），则第二个查询将改为：  
```
ps=10 field1:"foo bar"~10^50 OR field2:"foo bar"~10^20
```

这意味着如果在文档中出现的术语“foo”和“bar”之间的术语互不超过10个，那么这个短语就会匹配。例如文档显示：  
```
*Foo* term1 term2 term3 *bar*
```

将匹配短语查询。  
如何使用短语slop？通常它是在请求处理程序（in solrconfig）中配置的。  
使用查询slop（qs）的概念是相似的，但它适用于来自用户的显式短语查询。例如，如果您要搜索名称，则可以输入：  
```
q="Hans Anderson"
```

包含“汉斯·安德森”的文件将匹配，但包含中间名“基督徒”或名称是先写姓（“安德森，汉斯”）的文件不会。对于这些情况，可以配置查询字段qs，以便即使用户搜索明确的短语查询，也会应用slop。  
最后，除了fields（pf）参数这个短语外，edismax还支持pf2和pf3参数，用于创建bigram和trigram短语查询的字段。这些参数的查询语句slop可以分别使用ps2和ps3参数来指定。如果使用pf2/ pf3但ps2/ ps3，那么这些参数的查询语句会从ps参数中取出，如果有的话。  

## 使用 “Magic Fields”：_val_ 和 _query_

Solr 查询解析器中 _val_ 和 _query_ 的用法与 Lucene 查询解析器中的不同之处在于下列几处：  

    - 如果 magic 字段名称：_val_ 在术语或短语查询中使用，则将该值作为函数进行分析。
    - 它为 FunctionQuery 语法提供了一个 hook。包含括号的函数需要使用引号。例如：
```
_val_:myfield _val_:"recip(rord(myfield),1,2,3)"
```

    
    - Solr 查询解析器为任何类型的查询解析器（通过 QParserPlugin）提供了嵌套的查询支持。如果嵌套查询包含保留字符，则通常需要引用封装。例如：
```
_query_:"{!dismax qf=myfield}how now brown cow"
```

