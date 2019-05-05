## Solr分面搜索（Faceting）及其参数介绍 
<div class="content-intro view-box ">
## Faceting

Solr 分面是根据索引术语将搜索结果排列成类别。
      
  
向搜索者提供索引的术语，以及每个术语找到多少匹配文档的数字计数。分面使用户可以轻松地浏览搜索结果，缩小搜索结果的范围。  

## 通用的分面参数<a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#general-facet-parameters"/>

控制分面有两个通用参数。
      
  

    - facet 参数
          
        如果设置为<code>true</code>，则此参数在查询响应中启用 faceting 计数。如果设置为<code>false</code>，则为 blank 或丢失值，则此参数将禁用分面。除非此参数设置为<code>true</code>，否则下面列出的其他参数都不起作用。默认值为 blank（false）。  
    
    - facet.query 参数
          
        此参数允许您在 Lucene 默认语法中指定任意查询来生成 faceting 计数。  
        
            默认情况下，Solr 的 faceting 功能会自动确定字段的唯一术语，并返回每个术语的计数。使用<code>facet.query</code>，您可以覆盖此默认行为，并选择您要查看哪些术语或表达式计算。在典型的分面实现中，您将指定一些<code>facet.query</code>参数。此参数对基于数字范围的分面或基于前缀的分面特别有用。  
          
        
            您可以多次设置该<code>facet.query</code>参数，以指示应将多个查询用作单独的 faceting 约束。  
          
        
            如果您要以默认语法以外的语法使用 faceting 查询，请在 faceting 查询前加上查询符号的名称。例如，要使用假设的<code>myfunc</code>查询解析器，可以像这样设置<code>facet.query</code>参数：  
          
         
```
facet.query={!myfunc}name~fred
```

          
    


## 字段值分面参数

可以使用几个参数来根据指定字段中的索引项（term）来触发分面。  
在使用这些参数时，务必记住在 Lucene 中 “term” 是一个非常具体的概念：它与在任何分析发生后被索引的文本字段/值对相关。对于包含词干、小写或单词拆分的文本字段，所生成的 term 可能不是您所期望的。  
如果您希望 Solr 对完整的文本字符串执行分析（用于搜索）和分面，请使用架构中的 copyField 指令来创建该字段的两个版本：一个文本和一个字符串。确保两者都是：indexed="true"。（有关该 copyField 指令的更多信息，请参阅文档、字段和架构设计。）  
除非另有说明，否则下面的所有参数都可以在 per-field 的基础上来指定为 f.&lt;fieldname&gt;.facet.&lt;parameter&gt; 语法。  

    - facet.field 参数
          
        该<code>facet.field</code>参数标识应该被视为一个分面的字段。它迭代该字段中的每个 Term，并使用该 Term 作为约束生成一个 facet 计数。此参数可以在查询中多次指定以选择多个分面字段。  
        <div/>
        Note：如果您未将此参数设置为架构中的至少一个字段，则本节中介绍的其他任何参数都不会有任何影响。  
    
    - facet.prefix 参数
          
        这个<code>facet.prefix</code>参数限制了以给定的字符串前缀开头的那些 Term。这不会以任何方式限制查询，只会限制查询返回的分面。  
    
    - facet.contains 参数
          
        该<code>facet.contains</code>参数限制了包含给定子字符串的 term。这不会以任何方式限制查询，只会限制查询返回的分面。  
    
    - facet.contains.ignoreCase 参数
          
        如果使用<code>facet.contains</code>，则在将给定的子字符串与候选分面 term 匹配时，<code>facet.contains.ignoreCase</code>参数会导致大小写被忽略。  
    
    - facet.sort 参数
          
        此参数确定分面字段约束的排序。  
        
            这个参数有两个选项：  
          
        
            
                
                    <ul>
                        <li>count
                              
                            通过计数对约束进行排序（最高计数优先）。  
                        
                        - index
                              
                            返回按其索引顺序排序的约束（按索引 term 字典顺序）。对于 ASCII 范围中的 term，这将按字母顺序排序。  
                        
                    
                  
              
          
        
            如果<code>facet.limit</code>大于 0，则默认值为<code>count</code>，否则默认为<code>index</code>。  
          
    </li>
    <li>facet.limit 参数
          
        此参数指定应该为 facet 字段返回的约束计数的最大数量（基本上是返回的字段的分面数）。负值意味着 Solr 将返回无限数量的约束计数。默认值是<code>100</code>。  
    </li>
    <li>facet.offset 参数
          
        该<code>facet.offset</code>参数表示在允许分页的约束列表中的偏移量。默认值是<code>0</code>。  
    </li>
    <li>facet.mincount 参数
          
        该<code>facet.mincount</code>参数指定要包括在响应中的 facet 字段所需的最小计数。如果一个字段的计数低于最小值，则不返回字段的分面。默认值是<code>0</code>。  
    </li>
    <li>facet.missing 参数
          
        如果设置为<code>true</code>，则此参数指示除了 facet 字段的基于 Term 的约束之外，还应计算与查询匹配的所有结果的计数，但是该字段没有分面值的计算应该返回。默认值是<code>false</code>。  
    </li>
    <li>facet.method 参数
          
        该<code>facet.method</code>参数用于选择面对某个字段时 Solr 应使用的算法或方法的类型。  
        
            以下方法可用：  
          
        
            
                
                    
                        - enum
                              
                            枚举一个字段中的所有 term，计算匹配该 term 的文档与匹配查询的文档的集合交集。  
                            
                                建议使用此方法分割只具有少量不同值的 faceting 多值字段。每个文档的平均值数量无关紧要。  
                              
                        
                        - fc
                              
                            通过循环遍历与查询匹配的文档并对每个文档中出现的 term 求和来计算 facet 数。
                                  
                              
                            
                                如果该字段是多值的或被标记（根据<code>FieldType.isTokened()</code>），则这当前使用 UnInvertedField 缓存来实现。在缓存中查找每个文档以查看它包含的 term/值，并为每个值递增一个计数。  
                              
                            
                                此方法适用于字段的索引值数量较多但每个文档的值数量较少的情况。对于多值字段，使用混合方法，使用<code>filterCache</code>的 term 过滤器来匹配多个文档的 term。这些字母<code>fc</code>代表字段缓存。  
                              
                        
                        - fcs
                              
                            单值字符串字段的每段字段 faceting。启用<code>facet.method=fcs</code>并控制与<code>threads</code>本地参数一起使用的线程数。此参数允许分面在快速索引更改时速度更快。  
                        
                    
                    默认值是<code>fc</code>（除了使用<code>BoolField</code>字段类型的字段和<code>facet.exists=true</code>被请求时），因为它倾向于使用较少的内存，并且当字段在索引中具有许多独特的 term 时速度更快。
                          
                      
                  
              
          
    </li>
    <li>facet.enum.cache.minDf 参数
          
        此参数指示在确定该 term 的约束计数时应使用 filterCache 的最小文档频率（匹配 term 的文档的数量）。这只用于分面的<code>facet.method=enum</code>方法。
              
          
    </li>
    
        大于零的值会降低 filterCache 的内存使用量，但会增加查询处理所需的时间。如果在具有大量 term 的字段上分面，并且希望减少内存使用量，请尝试将此参数设置为介于<code>25</code>和<code>50</code>之间的值，然后运行一些测试。然后，根据需要优化参数设置。  
      
    
        默认值是<code>0</code>，导致 filterCache 被用于字段中的所有术语。  
      
    <li>facet.exists 参数
          
        要将分面数加 1，请指定<code>facet.exists=true</code>。该参数可以与<code>facet.method=enum</code>一起使用或者省略时使用。它只能用于非字典字段（如字符串）。这可能会加速大指数或高基数面值的分面。  
    </li>
    <li>facet.excludeTerms 参数
          
        如果您想从分面计数中删除 term，但将它们保留在索引中，则该<code>facet.excludeTerms</code>参数允许您执行此操作。  
    </li>
    <li>facet.overrequest.count 和 facet.overrequest.ratio 参数
          
        在某些情况下，在分布式 Solr 查询中为某以分面返回的“top”约束的准确性可以通过“过度请求”从每个单独的分片中得到期望的约束（即，facet.limit）的数量来提高。在这些情况下，每个分片默认都会被要求提供最高<code>10 + (1.5 * facet.limit)</code>限制。
              
          
        
            在某些情况下，根据您的文档如何在您的分片中进行分区以及您使用的<code>facet.limit</code>值，您可能会发现增加或减少 Solr 过多请求的数量是有利的。这可以通过设置<code>facet.overrequest.count</code>（默认为<code>10</code>）和<code>facet.overrequest.ratio</code>（默认为<code>1.5</code>）参数来实现。  
          
    </li>
    <li>facet.threads 参数
          
        此参数将导致加载分面中使用的基础字段与指定的线程数并行执行。指定 facet.threads=N，其中 N 是使用的最大线程数。  
        
            忽略这个参数或者指定线程数<code>0</code>不会产生任何线程，只会使用主请求线程。指定负数的线程将最多创建<code>Integer.MAX_VALUE</code>线程。  
          
    </li>
</ul>

## 范围分面（Range Faceting）


## <a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#range-faceting"/>

您可以在任何日期字段或支持范围查询的任何数字字段上使用“范围分面”。这对于将诸如价格之类的一系列范围查询（作为查询方面）拼接在一起是特别有用的。  

    - facet.range 参数
          
        该<code>facet.range</code>参数定义了 Solr 应该为其创建范围分面的字段。例如：  
         
```
facet.range=price&amp;facet.range=age
facet.range=lastModified_dt
```

          
        <div/>
    
    - facet.range.start 参数
          
        该<code>facet.range.start</code>参数指定范围的下限。您可以用每个字段的语法来指定此参数<code>f.&lt;fieldname&gt;.facet.range.start</code>。例如：  
         
```
f.price.facet.range.start=0.0&amp;f.age.facet.range.start=10
f.lastModified_dt.facet.range.start=NOW/DAY-30DAYS
```

          
        <div/>
    
    - facet.range.end 参数
          
        <code><font color="#000000" face="Verdana, Arial, Helvetica, sans-serif"><span style="white-space: normal; background-color: rgb(255, 255, 255);">该 </span></font>facet.range.end</code>参数指定范围的上限。您可以用每个字段的语法来指定此参数<code>f.&lt;fieldname&gt;.facet.range.end</code>。例如：  
         
```
f.price.facet.range.end=1000.0&amp;f.age.facet.range.start=99
f.lastModified_dt.facet.range.end=NOW/DAY+30DAYS
```

          
        <div/>
    
    - facet.range.gap 参数
          
        每个范围的跨度表示为要添加到下限的值。对于日期字段，这应该使用 DateMathParser 语法（如：<code>facet.range.gap=%2B1DAY …​ '+1DAY'</code>）来表示。您可以使用以下语法在每个字段的基础上指定此参数<code>f.&lt;fieldname&gt;.facet.range.gap</code>。例如：  
         
```
f.price.facet.range.gap=100&amp;f.age.facet.range.gap=10
f.lastModified_dt.facet.range.gap=+1DAY
```

          
        <div/>
    
    - facet.range.hardend 参数
          
        该<code>facet.range.hardend</code>参数是一个布尔参数，它指定 Solr 如何处理 facet.range.gap 不能在 facet.range.start 和 facet.range.end 之间平均分配的情况。  
        
            如果为<code>true</code>，则最后一个范围约束将<code>facet.range.end</code>值作为上限的。如果为<code>false</code>，则最后一个范围将有最小可能的上限，然后是<code>facet.range.end</code>，这个范围就是指定范围间隙的确切宽度。此参数的默认值为 false。  
          
        
            该参数可以使用<code>f.&lt;fieldname&gt;.facet.range.hardend</code>语法在每个字段的基础上指定。  
          
    
    - facet.range.include 参数
          
        默认情况下，用于计算<code>facet.range.start</code>和<code>facet.range.end</code>之间的范围分面的区域包括它们的下界的和排他性的上界。用<code>facet.range.other</code>参数定义的“before”范围是排他性的，“later”范围是包含性的。这个默认值，相当于下面的“lower”，不会导致在边界上重复计数。您可以使用<code>facet.range.include</code>参数来修改此行为，方法如下：  
        
            
                
                    <ul>
                        <li>
                            <code>lower</code>：所有基于 gap 的范围包括它们的下限。  
                        
                        - 
                            <code>upper</code>：所有基于 gap 的范围包括它们的上限。  
                        
                        - 
                            <code>edge</code>：即使未指定相应的 upper/lower 选项，第一个和最后一个 gap 范围也包括它们的边界界限（第一个的下限和第二个的上限）。  
                        
                        - 
                            <code>outer</code>：“before”和“later”范围将包括其边界，即使第一个或最后一个范围已经包含这些边界。  
                        
                        - 
                            <code>all</code>：包括所有选项：<code>lower</code>，<code>upper</code>，<code>edge</code>，和<code>outer</code>。  
                        
                    
                  
              
          
        
            您可以使用<code>f.&lt;fieldname&gt;.facet.range.include</code>语法在每个字段上指定此参数，并且可以多次指定它以指示多个选项。  
          
        <div/>
        为了确保您避免重复计算，不要同时选择<code>lower</code>和<code>upper</code>，不要选择<code>outer</code>，也不要选择<code>all</code>。  
    </li>
    <li>facet.range.other 参数
          
        该<code>facet.range.other</code>参数指定除了<code>facet.range.start</code>和<code>facet.range.end</code>之间的每个范围约束的计数外，还应计算这些选项的计数：  
        
            
                
                    
                        - 
                            <code>before</code>：所有具有字段值的记录都低于第一个范围的下限。  
                        
                        - 
                            <code>after</code>：所有具有字段值的记录都大于最后一个范围的上限。  
                        
                        - 
                            <code>between</code>：所有具有字段值的记录都在所有范围的开始和结束边界之间。  
                        
                        - 
                            <code>none</code>：不要计算任何计数。  
                        
                        - 
                            <code>all</code>：计算 before，between 和 later 的计数。  
                        
                    
                  
              
          
        
            这个参数可以用每个字段的<code>f.&lt;fieldname&gt;.facet.range.other</code>语法来指定，除了<code>all</code>选项外，这个参数可以被多次指定来指示多个选项，但是<code>none</code>会覆盖所有其他的选项。  
          
    </li>
    <li>facet.range.method 参数
          
        该<code>facet.range.method</code>参数选择 Solr 应用于范围分面的算法或方法的类型。这两种方法产生相同的结果，但性能可能会有所不同。  
        
            
                
                    
                        - 过滤
                              
                            此方法根据其他 facet.range 参数生成范围，并为每个参数执行一个过滤器，稍后与主查询结果集相交以获取计数。它将使用 filterCache，所以它将有足够大的缓存来包含所有的范围。  
                        
                        - DV
                              
                            此方法循环遍历与主查询匹配的文档，并为每个文档找到该值的正确范围。此方法将使用 docValues（如果为字段启用）或 fieldCache。该<code>dv</code>方法不支持字段类型DateRangeField 或使用 group.facets。  
                        
                    
                  
              
          
        
            该参数的默认值是<code>filter</code>。  
          
        日期范围和时区：分面上的日期字段是一个常见的情况，该[<code>TZ</code>](http://lucene.apache.org/solr/guide/7_0/working-with-dates.html#tz)参数可用于确保 "每天的分面计数" 或 "每月的分面计数" 基于有意义的定义，对于给定的日期/月份，"starts" 与特定时区相关。  
        有关详细信息, 请参阅 "使用日期" 部分中的示例。  
    </li>
</ul>

### 范围分面中的 facet.mincount <a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#facet-mincount-in-range-faceting"/>

该 facet.mincount 参数与字段分面中使用的相同，也适用于范围刻面。使用时，响应中将不包括低于最小值的范围。  

## 支点（决策树）分面<a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#pivot-decision-tree-faceting"/>

透视（Pivoting）是一种汇总工具，可让您自动对表中存储的数据进行排序、计数、汇总或平均值。结果通常显示在显示汇总数据的第二个表格中。Pivot 分面可以让您通过多个字段创建分面文档的结果汇总表。
      
  
另一种看待问题的方式是查询产生一个决策树，因为 Solr 告诉您“关于分面 A，约束/计数是：X/N，Y/M 等等。如果您用 X 来约束A，那么 B 的约束计数将是 S/P，T/Q等“。换句话说，如果从当前分面结果中应用一个约束，它会事先告诉您什么是“下一个”分面结果集。  

    - <span style="font-weight: normal;">facet.pivot 参数  

    
        该<code>facet.pivot</code>参数定义了用于 pivot 的字段。多个<code>facet.pivot</code>值将在响应中创建多个 “facet_pivot” 部分。用逗号分隔每个字段列表。  
    </span>
    
    - facet.pivot.mincount 参数
          
        该<code>facet.pivot.mincount</code>参数定义了为了将分面包括在结果中而需要匹配的文档的最小数目。默认值是 1。  
        
            使用 “bin / solr -e techproducts” 示例，像这样的查询 URL 将返回下面的数据，并在 “facet_pivot” 部分中找到 pivot 分面结果：  
```
http://localhost:8983/solr/techproducts/select?q=*:*&amp;facet.pivot=cat,popularity,inStock
   &amp;facet.pivot=popularity,cat&amp;facet=true&amp;facet.field=cat&amp;facet.limit=5&amp;rows=0&amp;facet.pivot.mincount=2
```

          
 
```
{  "facet_counts":{
    "facet_queries":{},
    "facet_fields":{
      "cat":[
        "electronics",14,
        "currency",4,
        "memory",3,
        "connector",2,
        "graphics card",2]},
    "facet_dates":{},
    "facet_ranges":{},
    "facet_pivot":{
      "cat,popularity,inStock":[{
          "field":"cat",
          "value":"electronics",
          "count":14,
          "pivot":[{
              "field":"popularity",
              "value":6,
              "count":5,
              "pivot":[{
                  "field":"inStock",
                  "value":true,
                  "count":5}]}]
}]}}}
```

    


### 统计组件与支点（Pivot）相结合


### <a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#combining-stats-component-with-pivots"/>

除了一些其他类型的分面所支持的一般局部参数外，一个 stats 局部参数可以和 facet.pivot 一起使用来指向 stats.field 实例（通过标签），即，您想计算每个支点约束（Pivot Constraint）。
      
  
在下面的示例中，为每个 facet.pivot 结果层次结构计算两个不同（重叠）的统计信息集合：  
```
stats=true
stats.field={!tag=piv1,piv2 min=true max=true}price
stats.field={!tag=piv2 mean=true}popularity
facet=true
facet.pivot={!stats=piv1}cat,inStock
facet.pivot={!stats=piv2}manu,inStock
```

结果如下：  
```
{"facet_pivot":{
  "cat,inStock":[{
      "field":"cat",
      "value":"electronics",
      "count":12,
      "pivot":[{
          "field":"inStock",
          "value":true,
          "count":8,
          "stats":{
            "stats_fields":{
              "price":{
                "min":74.98999786376953,
                "max":399.0}}}},
        {
          "field":"inStock",
          "value":false,
          "count":4,
          "stats":{
            "stats_fields":{
              "price":{
                "min":11.5,
                "max":649.989990234375}}}}],
      "stats":{
        "stats_fields":{
          "price":{
            "min":11.5,
            "max":649.989990234375}}}},
    {
      "field":"cat",
      "value":"currency",
      "count":4,
      "pivot":[{
          "field":"inStock",
          "value":true,
          "count":4,
          "stats":{
            "stats_fields":{
              "price":{
                "..."
  "manu,inStock":[{
      "field":"manu",
      "value":"inc",
      "count":8,
      "pivot":[{
          "field":"inStock",
          "value":true,
          "count":7,
          "stats":{
            "stats_fields":{
              "price":{
                "min":74.98999786376953,
                "max":2199.0},
              "popularity":{
                "mean":5.857142857142857}}}},
        {
          "field":"inStock",
          "value":false,
          "count":1,
          "stats":{
            "stats_fields":{
              "price":{
                "min":479.95001220703125,
                "max":479.95001220703125},
              "popularity":{
                "mean":7.0}}}}],
      "..."}]}}}}]}]}}
```


### 将 Facet 查询和 Facet 范围与 Pivot 分面相结合


### <a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#combining-facet-queries-and-facet-ranges-with-pivot-facets"/>

query 本地参数可以与 facet.pivot 一起使用，来指代 facet.query 应计算每个支点（pivot）约束的实例（按标签）。同样，range 本地参数也可以与 facet.pivot 一起使用以指向facet.range 实例。
      
  
在下面的例子中，两个查询分面用来计算 facet.pivot 结果层次结构的 h：  
```
facet=true
facet.query={!tag=q1}manufacturedate_dt:[2006-01-01T00:00:00Z TO NOW]
facet.query={!tag=q1}price:[0 TO 100]
facet.pivot={!query=q1}cat,inStock
```
```
{"facet_counts": {
    "facet_queries": {
      "{!tag=q1}manufacturedate_dt:[2006-01-01T00:00:00Z TO NOW]": 9,
      "{!tag=q1}price:[0 TO 100]": 7
    },
    "facet_fields": {},
    "facet_dates": {},
    "facet_ranges": {},
    "facet_intervals": {},
    "facet_heatmaps": {},
    "facet_pivot": {
      "cat,inStock": [
        {
          "field": "cat",
          "value": "electronics",
          "count": 12,
          "queries": {
            "{!tag=q1}manufacturedate_dt:[2006-01-01T00:00:00Z TO NOW]": 9,
            "{!tag=q1}price:[0 TO 100]": 4
          },
          "pivot": [
            {
              "field": "inStock",
              "value": true,
              "count": 8,
              "queries": {
                "{!tag=q1}manufacturedate_dt:[2006-01-01T00:00:00Z TO NOW]": 6,
                "{!tag=q1}price:[0 TO 100]": 2
              }
            },
            "..."]}]}}}
```

以类似的方式，在下面的示例中，为每个 facet.pivot 结果层次计算两个范围分面：  
```
facet=true
facet.range={!tag=r1}manufacturedate_dt
facet.range.start=2006-01-01T00:00:00Z
facet.range.end=NOW/YEAR
facet.range.gap=+1YEAR
facet.pivot={!range=r1}cat,inStock
```

结果如下：  
```
{"facet_counts":{
    "facet_queries":{},
    "facet_fields":{},
    "facet_dates":{},
    "facet_ranges":{
      "manufacturedate_dt":{
        "counts":[
          "2006-01-01T00:00:00Z",9,
          "2007-01-01T00:00:00Z",0,
          "2008-01-01T00:00:00Z",0,
          "2009-01-01T00:00:00Z",0,
          "2010-01-01T00:00:00Z",0,
          "2011-01-01T00:00:00Z",0,
          "2012-01-01T00:00:00Z",0,
          "2013-01-01T00:00:00Z",0,
          "2014-01-01T00:00:00Z",0],
        "gap":"+1YEAR",
        "start":"2006-01-01T00:00:00Z",
        "end":"2015-01-01T00:00:00Z"}},
    "facet_intervals":{},
    "facet_heatmaps":{},
    "facet_pivot":{
      "cat,inStock":[{
          "field":"cat",
          "value":"electronics",
          "count":12,
          "ranges":{
            "manufacturedate_dt":{
              "counts":[
                "2006-01-01T00:00:00Z",9,
                "2007-01-01T00:00:00Z",0,
                "2008-01-01T00:00:00Z",0,
                "2009-01-01T00:00:00Z",0,
                "2010-01-01T00:00:00Z",0,
                "2011-01-01T00:00:00Z",0,
                "2012-01-01T00:00:00Z",0,
                "2013-01-01T00:00:00Z",0,
                "2014-01-01T00:00:00Z",0],
              "gap":"+1YEAR",
              "start":"2006-01-01T00:00:00Z",
              "end":"2015-01-01T00:00:00Z"}},
          "pivot":[{
              "field":"inStock",
              "value":true,
              "count":8,
              "ranges":{
                "manufacturedate_dt":{
                  "counts":[
                    "2006-01-01T00:00:00Z",6,
                    "2007-01-01T00:00:00Z",0,
                    "2008-01-01T00:00:00Z",0,
                    "2009-01-01T00:00:00Z",0,
                    "2010-01-01T00:00:00Z",0,
                    "2011-01-01T00:00:00Z",0,
                    "2012-01-01T00:00:00Z",0,
                    "2013-01-01T00:00:00Z",0,
                    "2014-01-01T00:00:00Z",0],
                  "gap":"+1YEAR",
                  "start":"2006-01-01T00:00:00Z",
                  "end":"2015-01-01T00:00:00Z"}}},
                  "..."]}]}}}
```


### 额外的支点（Pivot）参数


### <a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#additional-pivot-parameters"/>

虽然 facet.pivot.mincount 名称与 facet.mincount 字段分面使用的参数有所不同，但上面介绍的许多分面参数也可用于 pivot 分面：  

    - facet.limit
    - facet.offset
    - facet.sort
    - facet.overrequest.count
    - facet.overrequest.ratio


## 区间分面<a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#interval-faceting"/>

另一种支持的分面形式是区间分面。这听起来类似于范围分面，但是功能更接近于使用范围查询进行分面查询。区间分面允许您设置可变区间并计算在指定字段中具有这些间隔内的值的文档的数量。
      
  
即使通过在范围查询中使用分面查询可以实现相同的功能，但这两种方法的实现是非常不同的，并且将根据上下文提供不同的性能。  
如果您担心搜索的性能，则应使用这两个选项进行测试。对于相同的字段，区间分面往往会有多个时间间隔，而在过滤器缓存更有效的环境（例如静态索引）中，分面查询倾向于更好。  
如果为该字段启用了区间分面，此方法将使用 docValues，否则将使用 fieldCache。  
使用这些参数进行区间分面：  
- facet.interval 参数  

  
        此参数指示必须应用区间分面的字段。它可以在同一请求中多次使用以指示多个字段。  
        
```
facet.interval=price&amp;facet.interval=size
```

          
    
- facet.interval.set 参数  

   
        此参数用于设置字段的时间间隔，可以多次指定，以指示多个时间间隔。这个参数是全局性的，这意味着它将被用于指定的所有字段，使用<code>facet.interval</code>，除非有特定字段的覆盖。如果要覆盖特定字段上的此参数，可以使用：<code>f.&lt;fieldname&gt;.facet.interval.set</code>例如：  
```
f.price.facet.interval.set=[0,10]&amp;f.price.facet.interval.set=(10,100]
```

    


### 区间语法<a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#interval-syntax"/>

区间必须以'（' 或 '[' 开头，然后是逗号（'，'），最终值，最后是 ')' 或']'。  
例如：  

    - （1,10） - 将包含大于1且小于10的值
    - [1,10] -  将包含大于或等于1且小于10的值
    - [1,10] -  将包含大于等于1且小于等于10的值

初始值和结束值不能为空。  
如果间隔需要是无限的，则特殊字符 * 可以用于开始和结束，限制。使用此特殊字符时，开始语法选项（(和[）和结束语法选项（)和]）将被视为相同。[*,*]将包括所有在该领域具有价值的文件。  
间隔限制可以是字符串，但不需要添加引号。所有的文本，直到逗号将被视为起始限制，并在此之后的文本将是最终限制。例如：[Buenos Aires,New York]。请记住，会进行类似字符串的比较来匹配字符串间隔中的文档（区分大小写）。比较器不能改变。  
逗号，括号和方括号可以通过在其前面使用 \ 而被转义。值之前和之后的空格将被省略。  
起始限制不能超过最终限制。等于限制是允许的，这允许您指示要计数的特定值：[A,A]，[B,B]和[C,Z]。  
区间分面支持下面介绍的输出键更换。输出键可以在 facet.interval 参数和 facet.interval.set 参数中被替换。例如：  
```
&amp;facet.interval={!key=popularity}some_field
&amp;facet.interval.set={!key=bad}[0,5]
&amp;facet.interval.set={!key=good}[5,*]
&amp;facet=true
```


## 分面的本地参数<a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#local-parameters-for-faceting"/>

该 LocalParams 语法允许压倒一切的全局设置。它也可以提供一种向其他参数值添加元数据的方法，就像 XML 属性一样。  

### 标记（Tagging）和排除（Excluding）过滤器


### <a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#tagging-and-excluding-filters"/>

您可以标记特定的过滤器，并在分面时排除这些过滤器。这在做多选分面时很有用。
      
  
请考虑以下具有分面的示例查询：  
```
q=mainquery&amp;fq=status:public&amp;fq=doctype:pdf&amp;facet=true&amp;facet.field=doctype
```

因为一切都已经被过滤器 doctype:pdf 约束了，所以 facet.field=doctypefacet 命令现在是多余的，并且除了 doctype:pdf 外一切都会返回 0 个计数。
      
  
为了实现 doctype 的多重选择方面，GUI 可能还想显示其他 doctype 类型值及其关联计数，就好像 doctype:pdf 约束还没有被应用。例如：  
```
=== Document Type ===
  [ ] Word (42)
  [x] PDF  (96)
  [ ] Excel(11)
  [ ] HTML (63)
```

要返回当前未选中的 doctype 值的计数，请直接约束 doctype 的标记筛选器，并在 DOCTYPE 上分面时排除这些过滤器。
      
  
```
q=mainquery&amp;fq=status:public&amp;fq={!tag=dt}doctype:pdf&amp;facet=true&amp;facet.field={!ex=dt}doctype
```

所有类型的分面都支持过滤排除。无论是 tag 和 ex 本地参数都可以通过用逗号分隔指定多个值。  

### 更改输出键<a href="http://lucene.apache.org/solr/guide/7_0/faceting.html#changing-the-output-key"/>

要更改分面命令的输出键，请使用 key 本地参数指定一个新名称。例如：  
```
facet.field={!ex=dt key=mylabel}doctype
```

上面的参数设置会导致 “doctype” 字段的字段分面结果在响应中使用键 “mylabel” 而不是 “doctype” 返回。当在不同的排除情况下多次对同一个字段进行排列时，这会很有帮助。  

### 限制某些 Term 的分面

要用某些 term 限制字段分面，请使用逗号分隔 terms 本地参数。逗号和引号可以用反斜线来转义，如 \,。在这种情况下，facet 的计算方式类似于 facet.method=enum 但忽略facet.enum.cache.minDf。例如：  
```
facet.field={!terms='alfa,betta,with\,with\',with space'}symbol
```
