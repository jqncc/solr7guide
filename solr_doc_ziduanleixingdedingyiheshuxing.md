## Solr字段类型的定义和属性 
<div class="content-intro view-box ">Solr 字段类型定义在对文档进行索引或向索引发送查询时将在字段上发生的分析。
      
  
字段类型定义可以包括以下四种类型的信息：  

    - 字段类型的名称（必填）。
    - 一个实现类的名字（必填）。
    - 如果一个字段的类型是 TextField，则为字段类型的字段分析说明。
    - 字段类型属性取决于实现类，一些属性可能是强制性的。


## schema.xml 中的字段类型定义<a href="http://lucene.apache.org/solr/guide/7_0/field-type-definitions-and-properties.html#field-type-definitions-in-schema-xml"/>

字段类型在 schema.xml 中定义。每个字段类型在 fieldType 元素之间定义。他们可以有选择地分组在一个 types 元素中。下面是一个名为 text_general 类型的字段类型定义的示例：
      
  
```
&lt;fieldType name="text_general" class="solr.TextField" positionIncrementGap="100"&gt; 
  &lt;analyzer type="index"&gt; 
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" /&gt;
    &lt;!-- in this example, we will only use synonyms at query time
    &lt;filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/&gt;
    --&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
  &lt;/analyzer&gt;
  &lt;analyzer type="query"&gt;
    &lt;tokenizer class="solr.StandardTokenizerFactory"/&gt;
    &lt;filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" /&gt;
    &lt;filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/&gt;
    &lt;filter class="solr.LowerCaseFilterFactory"/&gt;
  &lt;/analyzer&gt;
&lt;/fieldType&gt;
```

关于上述代码的提示信息：  

实现类负责确保字段被正确处理。在 schema.xml 类名中，字符串 solr 是 org.apache.solr.schema 或者 org.apache.solr.analysis 的简写形式。所以，solr.TextField 是真的 org.apache.solr.schema.TextField。
      
  

## 字段类型属性<a href="http://lucene.apache.org/solr/guide/7_0/field-type-definitions-and-properties.html#field-type-properties"/>

字段类型 class 决定了字段类型的大部分行为，但也可以定义可选的属性。例如，date 字段类型的以下定义就定义了两个属性，sortMissingLast 和 omitNorms。
      
  
```
&lt;fieldType name="date" class="solr.DatePointField"
           sortMissingLast="true" omitNorms="true"/&gt;
```

可以为给定字段类型指定的属性分为三个主要类别：
      
  

    - 特定于字段类型的类的属性。
    - 常规属性 Solr 支持任何字段类型。
    - 字段默认属性可以在字段类型上指定，这些字段将由使用此类型而不是默认行为的字段继承。


### 字段的一般属性<a href="http://lucene.apache.org/solr/guide/7_0/field-type-definitions-and-properties.html#general-properties"/>

这些是字段的一般属性：  

**name**

    
        fieldType 的名称。该值用于字段定义中的“类型”属性中。强烈建议名称仅包含字母数字或下划线字符，不能以数字开头。目前这不是严格执行的。
              
          
    
**class**

    
        用于存储和索引此类型数据的类名。请注意，您可以用 “solr” 作为前缀包含的类名称。Solr 会自动找出哪些软件包可以搜索这个类，这样<code>solr.TextField</code>就可以工作了。
              
          
    
    
        
            如果您使用的是第三方类，则可能需要具有完全限定的类名称。<code>solr.TextField</code>是的完全限定等效项是<code>org.apache.solr.schema.TextField</code>。  
          
    
**positionIncrementGap**

    
        对于多值字段，指定多个值之间的距离，这可以防止虚假词组匹配。
              
          
    
**autoGeneratePhraseQueries**

    
        对于文本字段。如果为<code>true</code>，Solr 将自动生成相邻词语的短语查询。如果为<code>false</code>，则术语必须用双引号括起来以作为短语处理。
              
          
    
**enableGraphQueries**

    
        对于文本字段，查询时使用[<code>sow=false</code>](http://lucene.apache.org/solr/guide/7_0/the-standard-query-parser.html#standard-query-parser-parameters)（这是<code>sow</code>参数的默认值）。对具有查询分析器的字段类型使用 true (默认值)，包括具有图形感知的筛选器，例如同义词图形过滤器和字符分隔符图形过滤器。
              
          
    
    
        对于既有查询分析器使用的字段类型，使用 false 包括在缺少某些标记 (例如，Shingle Filter) 时可以匹配文档的筛选器。  
    


**docValuesFormat**

    
        定义用于此类型的字段的自定义 DocValuesFormat。这要求具有架构感知的编解码器（例如，SchemaCodecFactory 已在 solrconfig.xml 中配置）。
              
          
    
**postingsFormat**
    
        定义用于此类型的字段的自定义 PostingsFormat。这要求具有架构感知的编解码器（例如，SchemaCodecFactory 已在 solrconfig.xml 中配置）。  
    
    
        Tip：只有默认编解码器支持 Lucene 索引反向兼容。如果您选择在 schema.xml 中自定义 postingsFormat 或 docValuesFormat，则升级到未来版本的 Solr 可能会要求您切换回默认编解码器，并在升级之前优化索引以将其重写为默认编解码器，或升级后从头开始重新构建整个索引。  
    


### 字段默认属性<a href="http://lucene.apache.org/solr/guide/7_0/field-type-definitions-and-properties.html#field-default-properties"/>

这些属性可以在字段类型中指定，也可以在单个字段中指定，以覆盖字段类型提供的值。
      
  
每个属性的默认值取决于底层的 FieldType 类，这又可能取决于的  &lt;schema/&gt; 的 version 属性。下面的表格包含了 FieldTypeSolr 提供的大多数实现的默认值，假设 schema.xml 声明 version="1.6"。  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
                    <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">属性</th>
            <th style="text-align: center;">描述</th>
            <th style="text-align: center;">值</th>
            <th style="text-align: center;">隐含默认值</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                索引（<span style="background-color: transparent;">indexed</span><span style="background-color: transparent;">）</span>
                  
            </td>
            <td>
                如果为 true，则可以在查询中使用该字段的值来检索匹配的文档。  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center;">true
                      
                  
            </td>
        </tr>
        <tr>
            <td>
                存储（<span style="background-color: transparent;">stored）</span>
                  
            </td>
            <td>
                如果为 true，则字段的实际值可以通过查询来检索。  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center;"><span style="background-color: transparent; text-align: left;">true</span>
                  
            </td>
        </tr>
        <tr>
            <td>
                docValues  
            </td>
            <td>
                如果为 true，则该字段的值将被放入一个面向列的 DocValues 结构中。  
            </td>
            <td>
                <p style="text-align: center; ">true 或者 <span style="background-color: transparent; text-align: left;">false</span>
                      
                  
            </td>
            <td>
                <p style="text-align: center; ">false
                      
                  
            </td>
        </tr>
        <tr>
            <td>
                sortMissingFirst   
                sortMissingLast  
            </td>
            <td>
                排序字段不存在时控制文档的位置。  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center; ">false
                      
                  
            </td>
        </tr>
        <tr>
            <td>
                多值（<span style="background-color: transparent;">multiValued</span><span style="background-color: transparent;">）</span>
                  
            </td>
            <td>
                如果为 true，则表示单个文档可能包含此字段类型的多个值。  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false  
            </td>
            <td style="text-align: center;">false</td>
        </tr>
        <tr>
            <td>
                omitNorms  
            </td>
            <td>
                如果为 true，则省略与该字段关联的规范（这将禁用该字段的长度规范化，并保存一些内存）。<span style="background-color: transparent;">对于所有基元 (non-analyzed) 字段类型</span><span style="background-color: transparent;">（如 int、float、data、bool 和 string）的默认值均为true。只有全文字段或字段需要规范。</span>
                  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center;">*  
            </td>
        </tr>
        <tr>
            <td>
                omitTermFreqAndPositions  
            </td>
            <td>
                <span style="background-color: transparent;">如果为 true</span><span style="background-color: transparent;">，则省略该字段过帐的术语频率、位置和有效载荷。这可以提高不需要这些信息的字段的性能。这也减少了索引所需的存储空间。依赖于使用此选项在字段上发布的位置的查询将悄然无法找到文档。对于不是文本字段的所有字段类型，此属性默认为 true。</span>
                  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center;">*  
            </td>
        </tr>
        <tr>
            <td>
                omitPositions  
            </td>
            <td>
                类似于 omitTermFreqAndPositions 但保留了词频信息。  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center;">*  
            </td>
        </tr>
        <tr>
            <td>
                termVectors termPositions termOffsets termPayloads  
            </td>
            <td>
                这些选项指示 Solr 维护每个文档的全部向量矢量，可选地包括这些向量中每个词条出现的位置，偏移和有效载荷信息。这些可以用来加速突出显示和其他辅助功能，但在索引大小方面会带来相当大的成本。对于 Solr 的典型用途，它们不是必需的。  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center;">false
                      
                  
            </td>
        </tr>
        <tr>
            <td>
                需要（<span style="background-color: transparent;">required</span><span style="background-color: transparent;">）</span>
                  
            </td>
            <td>
                指示 Solr 拒绝任何尝试添加一个文件，该文件没有这个字段的值。该属性默认为 false。  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center; ">false
                      
                  
            </td>
        </tr>
        <tr>
            <td>
                useDocValuesAsStored  
            </td>
            <td>
                如果该字段启用了 docValues，将其设置为 true 将允许 stored=false 在 fl 参数中匹配“*”时将该字段作为存储字段返回（即使有）。  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center;">true
                      
                  
            </td>
        </tr>
        <tr>
            <td>
                大（<span style="background-color: transparent;">large</span><span style="background-color: transparent;">）</span>
                  
            </td>
            <td>
                如果实际值&lt;512KB，大字段总是被延迟加载，并且只占用文档高速缓存中的空间。这个选项需要 stored="true" 和multiValued="false"。它的目的是为了可能有非常大的值，以便他们不被缓存在内存中的字段。  
            </td>
            <td>
                <p style="text-align: center;">true 或者 false
                      
                  
            </td>
            <td>
                <p style="text-align: center;">false
                      
                  
            </td>
        </tr>
    </tbody>
</table>

## 字段类型相似性<a href="http://lucene.apache.org/solr/guide/7_0/field-type-definitions-and-properties.html#field-type-similarity"/>

字段类型可以选择指定一个 &lt;similarity/&gt;，在对引用此类型的字段进行评分的文档时将使用它，只要集合的 "全局" 相似性允许这样做。
      
  
默认情况下，任何没有定义相似性的字段类型都会使用 BM25Similarity。有关更多详细信息以及配置全局和每类相似性的示例，请参阅其他架构元素。  
