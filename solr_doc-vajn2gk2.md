## Solr常用的查询参数 
<div class="content-intro view-box ">在 Solr 中几个查询解析器可以共享由 Solr 支持的查询参数。  
  
以下部分描述了 Solr 中常见的查询参数，Search RequestHandlers 支持这些参数。  

## defType 参数
defType 参数选择 Solr 应该用来处理请求中的主查询参数（q）的查询解析器。例如：  

```
defType=dismax
```
如果没有指定 defType 参数，则默认使用标准查询解析器。（如：defType=lucene）  

## sort 参数

sort 参数按升序 (asc) 或降序 (desc) 顺序排列搜索结果。该参数可以与数字或字母内容一起使用。方向可以全部以小写字母或全部大写字母输入（即，asc 或者ASC）。  
  
Solr 可以根据文档分数或具有单个值的任何字段的值对查询响应进行排序，该字段具有索引或使用 DocValues 的单个值（即任何字段，它在架构属性包括multiValued="false"，要么 docValues="true" 或 indexed="true"- 如果该字段没有启用 DocValues，则使用索引术语在运行时以动态方式生成它们），条件是：  

    - 该字段是非标记化的（即，该字段没有分析器，并且其内容已经被解析为标记，这会使排序不一致），或者
    - 该字段使用仅生成一个词的分析器（如 KeywordTokenizer）。

如果您希望能够对要标记其内容的字段进行排序以便于搜索，请使用架构中的  copyField 指令克隆该字段。然后在该字段上搜索并对其克隆进行排序。  
  
该表说明 Solr 如何响应 sort 参数的各种设置：  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">例</th>
            <th style="text-align: center;">结果</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td/>
            <td>
                如果省略了 sort 参数，则执行排序就好像将该参数设置为 score<code>desc</code>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center; ">score desc  
            </td>
            <td>
                从最高分到最低分按降序排列  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">price asc  
            </td>
            <td>
                按 price 字段的升序排序  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">inStock desc，price asc  
            </td>
            <td>
                按降序排列<code>inStock</code>字段<span style="background-color: transparent;">的内容，然后按照 price 字段的内容升序排序</span>  
            </td>
        </tr>
    </tbody>
</table>
关于 sort 参数的参数：  
  

    - 排序顺序必须包含一个字段名称（或作为伪字段的 score），后跟空格（在 URL 字符串中转义为 + 或 %20），然后是排序方向（asc 或 desc）。
    - 多个排序顺序可以用逗号隔开，使用下面的语法：
```
 sort=&lt;field name&gt;&lt;direction&gt;,&lt;field name&gt;&lt;direction&gt;],…​
```
如果提供了多个排序标准，则只有在第一个条目产生并列时才使用第二个条目。如果有第三个条目，则只有在第一个和第二个条目是并列的情况下才能使用。这种模式会在之后的条目中继续。  

## start 参数

指定时，start 参数指定查询结果集中的偏移量，并指示 Solr 开始显示此偏移量的结果。  
默认值是 0。换句话说，默认情况下，Solr 返回的结果没有偏移量，从结果开始的地方开始。  
将该 start 参数设置为某个其他数字（例如3，）会导致 Solr 跳过前面的记录，并从由偏移量标识的文档开始。  
您可以使用这个 start 参数来进行分页。例如，如果 rows 参数设置为10，则可以通过将 start 设置为0来显示3个连续的结果页面，然后重新发出相同的查询并将 start 设置为10，然后再次发出查询并将 start 设置为 20。  

## rows 参数

您可以使用该 rows 参数将查询的结果分页。该参数指定 Solr 应该一次返回到客户端的完整结果集中的最大文档数目。  
默认值是10。也就是说，默认情况下，Solr 一次返回 10 个文档以响应查询。  

## fq（Filter Query）参数

fq 参数定义了一个查询，可以用来限制可以返回的文档的超集，而不影响 score。这对于加快复杂查询非常有用，因为指定的查询 fq 是独立于主查询而被缓存的。当以后的查询使用相同的过滤器时，会有一个缓存命中，过滤器结果从缓存中快速返回。  
使用该 fq 参数时，请记住以下几点：  

    - 该 fq 参数可以在查询中多次指定。如果文档位于参数的每个实例所产生的文档集的交集中，则文档将仅包含在结果中。在下面的例子中，只有流行度大于10并且段落为0的文档才会匹配。
```
fq=popularity:[10 TO *]&amp;fq=section:0
```

    - filter 查询可能涉及复杂的 Boolean 查询。上面的例子也可以写成一个单独 fq 的两个强制性的子句，如下所示：
```
fq=+popularity:[10 TO *] +section:0
```

    - 每个过滤器查询的文档集都是独立缓存的。因此，关于前面的例子：如果这些条款经常出现在一起，则使用一个包含两个强制性条款的单个 fq，如果它们相对独立，则使用两个单独的 fq 参数。（要了解调整高速缓存大小并确保过滤器缓存是实际存在的，请参阅“良好配置的 Solr 实例”。）
    - 还可以在 fq 内部使用 filter(condition) 语法来单独缓存子句， 以及在其他情况下，实现缓存的筛选器查询的联合。
    - 与所有参数一样：URL 中的特殊字符需要正确转义并编码为十六进制值。在线工具可以帮助您使用 URL 编码。例如：http : //meyerweb.com/eric/tools/dencoder/。

## fl（Field List）参数

该 fl 参数将查询响应中包含的信息限制在指定的字段列表中。这些字段必须是 stored="true" 或 docValues="true"。  
  
字段列表可以指定为空格分隔或逗号分隔的字段名称列表。字符串“score”可以用来表示特定查询的每个文档的分数应该作为字段返回。通配符 * 选择文档中的所有字段，它们是 stored="true"、docValues="true" 和 useDocValuesAsStored="true"（当启用 docValues 时，这是默认字段）。您还可以添加伪字段（pseudo-fields）、函数和变换器到字段列表请求。  
本表显示了如何使用 fl 参数的一些基本示例：  
<table class="">
    <colgroup>
        <col/>
            <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">字段列表（Field List）</th>
            <th style="text-align: center;">结果</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center; "><span style="text-align: left;">id name price</span>  
  
            </td>
            <td>
                仅返回 ID，name 和 price 字段。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">id,name,price  
  
            </td>
            <td>
                仅返回 ID，name 和 price 字段。  
  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">id name, price  
  
            </td>
            <td>
                仅返回 ID，name 和 price 字段。  
  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;"> <span style="background-color: transparent; text-align: left;">id score</span>  
  
            </td>
            <td>
                返回 id 字段和 score。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">*  
            </td>
            <td>
                返回每个文档中的所有 stored 字段，以及任何 <span style="background-color: transparent;">useDocValuesAsStored="true"</span><span style="background-color: transparent;"> 的 </span><span style="background-color: transparent;">docValues </span><span style="background-color: transparent;">字段</span><span style="background-color: transparent;">。这是 fl 参</span><span style="background-color: transparent;">数的默认值。</span>  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">* score  
  
            </td>
            <td>
                返回每个文档中的所有字段以及每个字段的 score。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">*,dv_field_name  
  
            </td>
            <td>
                返回每个文档中的所有stored字段，以及任何具有 useDocValuesAsStored =“true” 的 docValues 字段和来自 dv_field_name 的 docValues，即使它具有useDocValuesAsStored =“false”。  
            </td>
        </tr>
    </tbody>
</table>
### 函数与 fl

    可以为结果中的每个文档计算函数，并将其作为伪字段（pseudo-field）返回：  
```
fl=id,title,product(price,popularity)
```

### 文件变换器与 fl

文档变换器可以用来修改查询结果中每个文档返回的信息：  
```
fl=id,title,[explain]
```

### 字段名称别名
您可以通过使用 “displayName” 前缀来更改对字段、函数或转换器的响应中使用的键。例如：  
```
fl=id,sales_price:price,secret_sauce:prod(price,popularity),why_score:[explain style=nl]
```
```
{
"response": {
    "numFound": 2,
    "start": 0,
    "docs": [{
        "id": "6H500F0",
        "secret_sauce": 2100.0,
        "sales_price": 350.0,
        "why_score": {
            "match": true,
            "value": 1.052226,
            "description": "weight(features:cache in 2) [DefaultSimilarity], result of:",
            "details": [{
                "..."
}]}}]}}
```

## debug 参数

该 debug 参数可以多次指定，并支持以下参数：  

    - debug=query：仅返回有关查询的调试信息。
    - debug=timing：返回有关查询花费多长时间处理的调试信息。
    - debug=results：返回关于 score 结果的调试信息（也称为“解释”）。默认情况下，score 解释以大字符串值的形式返回，对结构和可读性使用换行符和制表符缩进行，但是可以指定一个附加参数 debug.explain.structured=true 来将此信息作为 wt 请求的响应格式的嵌套数据结构返回。
    - debug=all：返回关于 request 请求的所有可用调试信息。（可替代地使用：debug=true）  

为了向后兼容老版本的 Solr，debugQuery=true 可以将其指定为另一种指示方式 debug=all。  
  
默认行为是不包含调试信息。  

## <span style="font-family: inherit;">explainOther </span>参数

该 explainOther 参数指定了一个 Lucene 查询来标识一组文档。如果包含此参数并设置为非空值，则查询将返回调试信息以及与 Lucene 查询相匹配的每个文档的“说明信息”（相对于主查询（由 q 指定）参数）。例如：  
```
q=supervillians&amp;debugQuery=on&amp;explainOther=id:juggernaut
```
上面的查询允许您检查顶级匹配文档的评分解释信息，将其与 id:juggernaut 文档匹配的解释信息进行比较，并确定排名不符合您的期望的原因。  
  
这个参数的默认值是空的，这不会导致返回额外的“解释信息”。  

## timeAllowed 参数

此参数指定允许搜索完成的时间量（以毫秒为单位）。如果此时间在搜索完成之前到期，任何部分结果将返回，但如 numFound、facet 数和结果的统计的值可能对整个结果集不准确。  
  
此值仅在以下时间检查：  
1 <li>查询扩展（Query Expansion）</li>2 <li>文件收集（）Document collection</li>由于此检查是周期性执行的，因此在中止请求之前处理请求的实际时间将略微大于或等于 timeAllowed 的值。如果请求在其他阶段中花费更多时间，自定义组件等，则不希望此参数中止请求。  

## segmentTerminateElely 参数

该参数可以设置为 true 或 false。  
如果设置为 true，并且如果此集合的 mergePolicyFactory 是 SortingMergePolicyFactory（使用的 sort 选项与此查询指定的 sort 参数兼容），则 Solr 将尝试使用 EarlyTerminatingSortingCollector。  
如果提前终止（early termination）使用，一个 segmentTerminatedEarly 标题将包含在 responseHeader。  
使用类似的 timeAllowed `Parameter, 当早期段终止发生时，例如值 `numFound，Facet 计数，并导致 Stats 可能不准确对整个结果集。  
  
这个参数的默认值是 false。  

## omitHeader 参数

该参数可以设置为 true 或 false。  
如果设置为 true，则此参数将从返回的结果中排除标题。标题包含有关请求的信息，例如完成所需的时间。该参数的默认值是 false。  

## wt 参数

该 wt 参数选择 Solr 应该用来格式化查询响应的 Response Writer。有关响应写入程序的详细说明，请参阅响应写入程序。  
如果您没有在查询中定义 wt 参数，那么 JSON 将作为响应的格式返回。  

## <span style="font-size: 14px;"> </span><span style="font-size: 14px;">cache </span>参数

Solr 默认缓存所有查询的结果并过滤查询。要禁用结果缓存，请设置 cache=false 参数。  
您也可以使用该 cost 选项来控制计算非缓存筛选器查询的顺序。这使您可以在昂贵的非缓存过滤器之前订购更便宜的非缓存过滤器。  
对于成本非常高的过滤器，如果 cache=falseand 并且 cost&gt;=100 和查询实现了 PostFilter 接口，则将从该查询请求收集器，并在匹配主查询和所有其他过滤器查询后用于过滤文档。可以有多个后置过滤器；他们也按成本排序。  
例如：  
这是一个正常的函数范围查询，用作过滤器，所有匹配的文件都是预先生成和缓存的：  
```
fq={!frange l=10 u=100}mul(popularity,price)
```
这是一个与传统的 lucene 过滤器并行运行的函数范围查询：  
```
fq={!frange l=10 u=100 cache=false}mul(popularity,price)
```
这是在每个已经匹配查询和所有其他过滤器的文档之后检查的函数范围查询。这对于非常昂贵的函数查询是很好的：  
```
fq={!frange l=10 u=100 cache=false cost=100}mul(popularity,price)
```

## logParamsList 参数

默认情况下，Solr 记录请求的所有参数。设置此参数以限制请求的哪些参数被记录。这可能有助于将日志记录控制为仅对贵组织认为重要的参数。  
例如，你可以像这样定义：  
```
logParamsList=q,fq
```
只有 'q' 和 'fq' 参数会被记录。  
如果没有参数应该被记录，你可以发送 logParamsList 为空（即，logParamsList=）。  
Tip：这个参数不仅适用于查询请求，而且适用于 Solr 的任何类型的请求。  

## echoParams 参数

该 echoParams 参数控制响应头中包含的有关请求参数的信息。  
该 echoParams 参数接受以下值：  
- explicit：这是默认值。只有实际请求中包含的参数以及 _参数（这是一个 64 位数字时间戳）将被添加到响应头的 params 部分。
    - all：包含对查询作出贡献的所有请求参数。这将包括在 solrconfig.xml 中找到的请求处理程序定义中定义的所有内容以及请求中包含的参数以及 _参数。如果参数包含在请求处理程序定义和请求中，则它将在响应头中出现多次。
    - none：完全删除响应头的 “params” 部分。在响应中没有关于请求参数的信息。

下面是一个 JSON 响应的例子，其中没有包含 echoParams 参数，所以缺省值 explicit 是活动的。创建此响应的请求的 URL 包括三个参数 - q，wt 和 indent：  
```
{
  "responseHeader": {
    "status": 0,
    "QTime": 0,
    "params": {
      "q": "solr",
      "indent": "true",
      "wt": "json",
      "_": "1458227751857"
    }
  },
  "response": {
    "numFound": 0,
    "start": 0,
    "docs": []
  }
}
```
如果发送了一个类似的请求，并添加 echoParams=all 到前面示例中使用的三个参数中，则会发生这种情况：  
```
{
  "responseHeader": {
    "status": 0,
    "QTime": 0,
    "params": {
      "q": "solr",
      "df": "text",
      "preferLocalShards": "false",
      "indent": "true",
      "echoParams": "all",
      "rows": "10",
      "wt": "json",
      "_": "1458228887287"
    }
  },
  "response": {
    "numFound": 0,
    "start": 0,
    "docs": []
  }
}
```
