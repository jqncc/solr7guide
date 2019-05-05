## Solr查询：DisMax查询解析器 
<div class="content-intro view-box ">Solr 的 DisMax 查询解析器设计用于处理由用户输入的简单短语（没有复杂的语法），并基于每个字段的重要性使用不同的权重（boosts）在多个字段中搜索各个术语。其他选项使用户能够根据每个用例的具体规则影响评分（独立于用户输入）。  
  
一般来说，DisMax 查询解析器的接口更像 Google 的接口，而不是 “lucene” Solr 查询解析器的接口。这种相似性使得 DisMax 成为许多使用者应用程序的合适查询解析器。它接受简单的语法，很少产生错误消息。  
DisMax 查询解析器支持 Lucene QueryParser 语法的一个极其简化的子集。和 Lucene 一样，引号可以用来对短语进行分组，+/- 可以用来表示强制性和可选的子句。所有其他的 Lucene 查询解析器特殊字符（AND 和 OR 除外）都被转义以简化用户体验。DisMax 查询解析器负责使用布尔子句构建用户输入的良好查询，该查询包含跨域的 DisMax 查询以及用户指定的提升。它还可以让 Solr 管理员提供额外的 boosting 查询、boosting 函数和过滤查询，以人为影响所有搜索的结果。这些选项都可以指定为 solrconfig.xml 文件中的请求处理程序的默认参数，或者在 Solr 查询 URL 中被重写。  
对 DisMax 名称背后的技术概念感兴趣？其实 DisMax 代表的是 Maximum Disjunction。以下是对 Maximum Disjunction 或 “DisMax” 查询的定义：  
```
一个查询，它生成由其子查询产生的文档的联合，该查询将为每个文档评分（由任何子查询生成的该文档的最高分数），并为任何其他匹配的子查询加一个并列中断增量。
```

不管您是否记得这个解释，记住 DisMax 查询解析器的主要目的是易于使用，并且几乎可以接受任何输入而不会返回错误。  

## DisMax 查询解析器参数<a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#dismax-query-parser-parameters"/>

除了常见的请求参数、突出显示参数和简单的 facet 参数外，DisMax 查询解析器还支持下面描述的参数。与标准查询解析器一样，DisMax 查询解析器允许在 solrconfig. xml 中指定默认参数值，或者在请求中由查询时间值重写。  
  
以下部分详细解释这些参数。  

### q 参数<a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#q-parameter"/>

该 q 参数定义了构成搜索本质的主要 “查询”。该参数支持用户提供的原始输入字符串，没有特殊的转义。术语中的 + 和 - 字符被视为“强制性”和“禁止”修饰符。用平衡的引号字符（例如，“San Jose”）包裹的文本被视为一个短语。包含奇数个引号字符的任何查询的计算方式就像根本没有引号字符一样。  
  
该 q 参数不支持通配符，如 *。  
  

### q.alt 参数<a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#q-alt-parameter"/>

如果指定，q.alt 参数定义一个查询（默认情况下将使用标准查询解析语法进行解析），当主 q 参数未指定或为空时。q.alt 当您需要一个类似于查询的东西来匹配所有的文档时（不要忘记 &amp;rows=0），q.alt 参数就派上用场了，可以获得收集范围的 faceting 计数。  
  

### qf（Query Fields）参数<a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#qf-query-fields-parameter"/>

qf 参数引入了一个字段列表，每个字段都分配了一个 boost 因子来增加或减少该字段在查询中的重要性。例如，下面的查询：  
  
```
qf="fieldOne^2.3 fieldTwo fieldThree^0.4"
```
分配 fieldOne 一个 2.3 的提升， fieldTwo 为默认提升（因为没有 boost 因素被指定），并且 fieldThree 提高 0.4。这些 boost 因素使 fieldOne 中的匹配比 fieldTwo 的匹配更加重要，它比 fieldThree 中的匹配更为重要。  

### <span style="font-family: inherit;">mm</span>（最小匹配）参数<a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#mm-minimum-should-match-parameter"/>

在处理查询时，Lucene / Solr 识别三种类型的子句：强制性的，禁止的和“optional”（也被称为“should”子句）。默认情况下，q 参数中指定的所有单词或短语都被视为“optional”子句，除非它们前面有“+”或“ - ”。处理这些“optional”子句时，该 mm 参数可以说这些子句的某些最小值必须匹配。DisMax 查询解析器在如何指定最小数字方面提供了很大的灵活性。  
  
下表解释了可以指定 mm 值的各种方法。  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">句法</th>
            <th style="text-align: center;">示例</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">正整数（<span style="background-color: transparent; text-align: left;">Positive integer</span><span style="background-color: transparent;">）</span>  
            </td>
            <td>
                <p style="text-align: center;">3  
            </td>
            <td>
                定义必须匹配的最小子句数，而不管总共有多少个子句。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">负整数（<span style="background-color: transparent; text-align: left;">Negative integer）</span>  
            </td>
            <td>
                <p style="text-align: center;">-2  
            </td>
            <td>
                将匹配子句的最小数量设置为可选子句的总数减去此值。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">百分比（<span style="background-color: transparent; text-align: left;">Percentage</span><span style="background-color: transparent;">）</span>  
            </td>
            <td>
                <p style="text-align: center;">75％  
            </td>
            <td>
                将匹配子句的最小数量设置为可选子句总数的百分比。从百分比计算出的数字向下取整，作为最小值。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">负比例（<span style="background-color: transparent; text-align: left;">Negative percentage</span><span style="background-color: transparent;">）</span>  
            </td>
            <td>
                <p style="text-align: center;">-25％  
            </td>
            <td>
                表示可选子句总数的百分比可能会丢失。从百分比计算出的数字向下舍入，然后从总数中减去以确定最小数量。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">表达式以正整数开头，后跟一个&gt;或&lt;符号和另一个值  
            </td>
            <td>
                <p style="text-align: center;">3 &lt;90％  
            </td>
            <td>
                定义一个条件表达式，指出如果可选子句的数目等于（或小于）整数，则它们都是必需的，但如果它大于整数，则应用规范。在这个例子中：如果有1到3个子句，他们都是必需的，但是对于4个或更多子句，只需要90％。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">涉及&gt;或&lt;符号的多个条件表达式  
            </td>
            <td>
                <p style="text-align: center;">2 &lt;-25％9 &lt;-3  
            </td>
            <td>
                定义了多个条件，每个条件<span style="background-color: transparent;">只对比它前面的一个更大的数字有效</span><span style="background-color: transparent;">。在左边的例子中，如果有1或2个子句，那么这两个都是必需的。如果有3-9个子句，只有25％是必需的。如果有9个以上的话，除了3个外都是必需的。</span>  
            </td>
        </tr>
    </tbody>
</table>
指定 mm 值时，请记住以下几点：  

    - 在处理百分比时，负值可用于在边缘情况下获得不同的行为。处理 4 个子句时，75％ 和-25％意味着同样的事情，但是当处理5个子句时，75％意味着需要3个，但是-25％意味着需要4个。
    - 如果基于参数的参数的计算确定不需要可选子句，那么有关布尔查询的常规规则仍然适用于搜索时间。（即，不包含必需子句的布尔查询仍然必须至少匹配一个可选子句）。
    - 无论计算到达的数量是多少，Solr 都不会使用大于可选子句数量的值，或者小于1的值。换句话说，无论计算结果多么低或多高，所需的最小数量匹配永远不会少于1或大于子句的数量。
    - 在跨不同查询分析器配置的多个字段进行搜索时，各个字段之间的可选子句数量可能不同。在这种情况下，由 mm 指定的值适用于可选子句的最大数量。例如，如果查询子句被视为其中一个字段的停用词，那么该字段的可选子句数量将小于其他字段的数量。如果将 mm 设置为100％，则带有这样一个停用词子句的查询将不会返回该字段中的匹配项，因为已删除的子句不会被计算为匹配项。

mm 的默认值为 100% (意味着所有子句都必须匹配)。  
  

### pf（Phrase Fields）参数

### <a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#pf-phrase-fields-parameter"/>

一旦使用 fq 和 qf 参数确定了匹配文档的列表，就可以使用 pf 参数“boost”文档的得分，因为 q 参数中的所有项都出现在非常接近的情况下。  
  
该格式与 qf 参数所使用的格式相同：当从整个 q 参数中进行短语查询时，字段列表和“boosts”将与每个字段相关联。  

### ps（Phrase Slop）参数
ps 参数指定应用于使用 pf 参数指定的查询的 “短语 slop” 的数量。短语 slop 是一个标记需要相对于另一个标记移动以匹配查询中指定的短语的位置的数量。  
  

### qs（Query Phrase Slop）参数<a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#qs-query-phrase-slop-parameter"/>

qs 参数指定用 qf 参数明确包含在用户查询字符串中的短语查询所允许的倾斜量。如上所述，slop 是指为了匹配在查询中指定的短语，一个标记需要相对于另一个标记移动的位置的数量。  

### tie（Tie Breaker）参数<a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#the-tie-tie-breaker-parameter"/>

tie 参数指定一个浮点值（应该远远小于1），以便在 DisMax 查询中用作 tiebreaker。  
  
当来自用户输入的术语针对多个字段进行测试时，可能会有多个字段匹配。如果是这样，每个字段将根据该字段在该字段中的普遍程度（对于每个文档相对于所有其他文档）产生不同的 score。通过该 tie 参数，您可以控制查询的最终 score 与最高 score 字段相比，将会受较低评分字段影响的程度。  
值为“0.0”（默认值）使查询成为纯粹的“分离最大查询”：也就是说，只有最大 scoring 子查询才有助于最终 score。值为“1.0”使查询成为一个纯粹的“分离总和查询”，因为最终 score 将是子查询 score 的总和，这与最大 scoring 子查询无关。通常，一个较低的值（如0.1）是有用的。  

### bq（Boost Query）参数

### <a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#bq-boost-query-parameter"/>

bq 参数指定一个附加的可选查询子句，将添加到用户的主要查询中以影响 score。例如，如果您想为最近的文档添加相关性 boost：  
```
q=cheese
bq=date:[NOW/DAY-1YEAR TO NOW/DAY]
```
您可以指定多个 bq 参数。如果您希望将查询作为单独的子句进行分析，请使用多个 bq 参数。  

### bf（Boost Functions）参数

### <a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#bf-boost-functions-parameter"/>

bf 参数指定将用于构造 FunctionQueries 的函数（具有可选的 boosts），该函数将作为将影响 score 的可选子句添加到用户的主查询中。可以使用由 Solr 本地支持的任何函数，以及 boost 值。例如：   
```
recip(rord(myfield),1,2,3)^1.5
```
使用 bf 参数指定函数本质上只是使用 bqparam 与 {!func} 解析器结合的简写。  
例如，如果要首先显示最近的文档，则可以使用以下任一项：  
```
bf=recip(rord(creationDate),1,1000,1000)
  ...or...
bq={!func}recip(rord(creationDate),1,1000,1000)
```

## 查询提交给 DisMax Query Parser 的示例<a href="http://lucene.apache.org/solr/guide/7_0/the-dismax-query-parser.html#examples-of-queries-submitted-to-the-dismax-query-parser"/>

本节中的所有示例 URL 均假设您正在运行 Solr 的 “techproducts” 示例：  
```
bin/solr -e techproducts
```
使用标准查询解析器得到单词 “video”的结果，我们假设 “df” 指向要搜索的字段：  
```
http://localhost:8983/solr/techproducts/select?q=video&amp;fl=name+score
```
"dismax" 解析器被配置为跨 text、features、name、sku、id、manu 和 cat 字段进行搜索，它们都具有不同的增强功能，旨在确保 "better" 的匹配首先出现，具体来说: 在 name 和 cat 字段上匹配的文档获得更高的分数。  
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=video
```
请注意，此实例还配置了一个默认的字段列表，可以在 URL 中重写。  
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=video&amp;fl=*,score
```
您还可以覆盖搜索的字段以及每个字段得到的 boost 程度。  
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=video&amp;qf=features^20.0+text^0.3
```
您可以提高具有与特定值匹配的字段的结果。  
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=video&amp;bq=cat:electronics^5.0
```
另一个请求处理程序在 “/ instock” 处注册，并具有稍微不同的配置选项，特别是：（inStock:true 的过滤器)。  
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=video&amp;fl=name,score,inStock
```
```
http://localhost:8983/solr/techproducts/instock?defType=dismax&amp;q=video&amp;fl=name,score,inStock
```
这个解析器中其他非常酷的功能之一是强大的支持，根据用户查询中有多少术语来指定要使用的“BooleanQuery.minimumNumberShouldMatch”。这些允许错别字和部分匹配的灵活性。对于 dismax 解析器，一个和两个单词查询要求所有可选子句匹配，但对于三到五个字查询，允许一个丢失的单词。  
  
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=belkin+ipod
```
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=belkin+ipod+gibberish
```
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=belkin+ipod+apple
```
使用 debugQuery 选项可以查看已解析的查询以及每个文档的分数解释。  
  
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=belkin+ipod+gibberish&amp;debugQuery=true
```
```
http://localhost:8983/solr/techproducts/select?defType=dismax&amp;q=video+card&amp;debugQuery=true
```
