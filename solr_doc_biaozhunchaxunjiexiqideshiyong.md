## Solr标准查询解析器的使用 
<div class="content-intro view-box ">Solr 的标准查询解析器（Query Parser）也被称为 “lucene” 解析器。
      
  
标准查询解析器的关键优势在于它支持强大且相当直观的语法，允许您创建各种结构化查询。最大的缺点是它不容忍出现语法错误，与 DisMax 查询解析器相比， DisMax 查询解析器的设计目的是尽可能地减少抛出错误。  

## 标准查询解析器参数

除了常见 Query 参数，Faceting 参数，Highlighting 显示参数之外，标准查询解析器还支持下列所述的参数。
      
  

**q**

    
        使用标准查询语法定义查询。该参数是强制性的。  
    
**q.op**

    
        指定查询表达式的默认运算符，覆盖在 Schema 中指定的默认运算符。可能的值是“AND”或“OR”。  
    
**df**

    
        指定默认字段，覆盖架构中默认字段的定义。  
    
**sow**

    
        拆分空白。如果设置为<code>true</code>，则对每个单独的空格分隔的术语分别调用文本分析。默认是<code>false</code>；空格分隔的术语序列将一次性提供给文本分析，使分析过滤器在术语序列上操作的适当功能，例如多词同义词和带状疱疹。  
    

默认参数值是在 solrconfig 中指定的，也可由请求中的查询时间值重写。
      
  

## 标准查询解析器响应

默认情况下，来自标准查询解析器的响应包含一个 &lt;result&gt; 块，这个块是未命名的。如果使用该 debug 参数，则将使用 "debug" 名称返回一个附加的 &lt;lst&gt; 块。这将包含有用的调试信息，包括原始查询字符串、解析的查询字符串以及 &lt;result&gt; 块中每个文档的解释信息。如果 explainOther 参数也被使用，那么将为所有匹配该查询的文档提供额外的解释信息。
      
  

### 响应示例

本节介绍标准查询解析器的响应示例。  
下面的 URL 提交一个简单的查询，并请求 XML Response Writer 使用缩进来使 XML 响应更具可读性。  

```
http://localhost:8983/solr/techproducts/select?q=id:SP2514N
```

结果如下：  
```
&lt;response&gt;
&lt;responseHeader&gt;&lt;status&gt;0&lt;/status&gt;&lt;QTime&gt;1&lt;/QTime&gt;&lt;/responseHeader&gt;
&lt;result numFound="1" start="0"&gt;
 &lt;doc&gt;
  &lt;arr name="cat"&gt;&lt;str&gt;electronics&lt;/str&gt;&lt;str&gt;hard drive&lt;/str&gt;&lt;/arr&gt;
  &lt;arr name="features"&gt;&lt;str&gt;7200RPM, 8MB cache, IDE Ultra ATA-133&lt;/str&gt;
    &lt;str&gt;NoiseGuard, SilentSeek technology, Fluid Dynamic Bearing (FDB) motor&lt;/str&gt;&lt;/arr&gt;
  &lt;str name="id"&gt;SP2514N&lt;/str&gt;
  &lt;bool name="inStock"&gt;true&lt;/bool&gt;
  &lt;str name="manu"&gt;Samsung Electronics Co. Ltd.&lt;/str&gt;
  &lt;str name="name"&gt;Samsung SpinPoint P120 SP2514N - hard drive - 250 GB - ATA-133&lt;/str&gt;
  &lt;int name="popularity"&gt;6&lt;/int&gt;
  &lt;float name="price"&gt;92.0&lt;/float&gt;
  &lt;str name="sku"&gt;SP2514N&lt;/str&gt;
 &lt;/doc&gt;
&lt;/result&gt;
&lt;/response&gt;
```

这里是一个有限的字段列表查询的例子。  
```
http://localhost:8983/solr/techproducts/select?q=id:SP2514N&amp;fl=id+name
```

结果如下：  
```
&lt;response&gt;
&lt;responseHeader&gt;&lt;status&gt;0&lt;/status&gt;&lt;QTime&gt;2&lt;/QTime&gt;&lt;/responseHeader&gt;
&lt;result numFound="1" start="0"&gt;
 &lt;doc&gt;
  &lt;str name="id"&gt;SP2514N&lt;/str&gt;
  &lt;str name="name"&gt;Samsung SpinPoint P120 SP2514N - hard drive - 250 GB - ATA-133&lt;/str&gt;
 &lt;/doc&gt;
&lt;/result&gt;
&lt;/response&gt;
```


## 指定标准查询解析器的术语

对标准查询解析器的查询被分解为术语和运算符。有两种类型的术语：单个术语和短语。  
  

    - 一个词是一个单词，如 “test” 或 “hello”
    - 短语是由双引号包围的一组单词，例如 “hello dolly”

可以将多个术语与布尔运算符组合在一起，形成更复杂的查询（如下所述）。  
重要的是，用于查询的解析器分析术语和短语的方式与用于索引分析术语和短语的分析器的方式一致；否则，搜索可能会产生意想不到的结果。  

### 术语修饰符

Solr 支持各种术语修饰符，根据需要为搜索添加灵活性或精确性。这些修饰符包括通配符，用于进行搜索的字符 “fuzzy” 或更一般化，等等。以下各节详细介绍了这些修饰符。  

### 通配符搜索

Solr 的标准查询解析器支持单个和多个字符通配符搜索。通配符可以应用于单项，但不能搜索短语。  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">通配符搜索类型</th>
            <th style="text-align: center;">特殊字符</th>
            <th style="text-align: center;">示例</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">单个字符（匹配单个字符）  
            </td>
            <td>
                <p style="text-align: center;">？  
            </td>
            <td>
                搜索字符串<code>te?t</code>将匹配 <span style="background-color: transparent;">test </span><span style="background-color: transparent;">和 </span><span style="background-color: transparent;">text</span><span style="background-color: transparent;">。</span>
                  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">多个字符（匹配零个或多个连续字符）  
            </td>
            <td>
                <p style="text-align: center;">*  
            </td>
            <td>
                通配符搜索：<code>tes*</code>将匹配 <span style="background-color: transparent;">test</span><span style="background-color: transparent;">，</span><span style="background-color: transparent;">testing </span><span style="background-color: transparent;">和 </span>
                    <span style="background-color: transparent;">tester</span><span style="background-color: transparent;">器。您也可以在术语中间使用通配符。例如：</span><code>te*t</code><span style="background-color: transparent;">会匹配 </span><span style="background-color: transparent;">test</span><span style="background-color: transparent;"> </span>
                        <span style="background-color: transparent;">和 </span><span style="background-color: transparent;">text</span><span style="background-color: transparent;">。</span><code>*est</code><span style="background-color: transparent;">会匹配 </span><span style="background-color: transparent;">pest </span>
                            <span style="background-color: transparent;">和 </span><span style="background-color: transparent;">test</span><span style="background-color: transparent;">。</span>
                  
            </td>
        </tr>
    </tbody>
</table>

### 模糊搜索

Solr 的标准查询解析器支持基于 Damerau-Levenshtein Distance 或 Edit Distance 算法的模糊搜索。模糊搜索发现与指定术语相似的术语，而不一定完全匹配。要进行模糊搜索，请在单词词尾添加波浪号 〜 符号。例如，要搜索与 “roam” 拼写相似的术语，请使用模糊搜索：
      
  
```
roam~
```

这个搜索将匹配像 roams、foam、foams。它也将匹配“roam”这个词本身。  
可选的距离参数指定允许编辑的最大数量，介于0和2之间，默认为2。例如：  
```
roam~1
```

这将匹配 roams 和 foam 等术语，但不包括 foams，因为它的编辑距离为“2”。  
在许多情况下，词干 (将术语减少到一个公共词干) 会对模糊搜索和通配符搜索产生类似的效果。
      
  

### 邻近搜索

邻近搜索查找介于特定距离之间的术语。  
如果要执行邻近搜索，请在搜索短语的末尾添加波浪号字符 〜 和数字值。例如，要在文档中搜索彼此相隔10个字的 “apache” 和 “jakarta”，请使用以下搜索：  
```
"jakarta apache"~10
```

这里所指的距离是匹配指定词组所需的术语移动次数。在上面的例子中，如果 “apache” 和 “jakarta” 在一个字段中相隔10个空格，而 “apache” 在 “jakarta” 之前出现，则需要10个以上的术语来移动这些术语并将 “apache” 定位在 "jakarta" 的右边，并在两者之间有一个空间。  

### 范围搜索

范围搜索指定字段的值范围（具有上限和下限的范围）。该查询与指定字段或字段的值位于该范围内的文档匹配。范围查询可以包含或不包括上限和下限。排序按字典顺序完成，数字字段除外。例如，下面的范围查询匹配其 popularity 字段的值在52和10000之间（包含）的所有文档：
      
  
```
popularity:[52 TO 10000]
```

范围查询不限于日期字段，甚至还包括数字字段。您也可以使用非日期字段的范围查询：  
```
title:{Aida TO Carmen}
```

这将查找所有标题介于 Aida 和 Carmen 之间的文件，但不包括 Aida 和 Carmen 的文件。  
查询周围的括号决定了它的包容性。  

    - 方括号 [＆] 表示一个包含范围查询，它匹配包括上限和下限在内的值。
    - 大括号{＆}表示一个独占范围查询，它匹配上限和下限之间的值，但不包括上边界和下限本身。
    - 您可以混合使用这些类型，因此范围的一端是包含性的，另一端是独占的。这是一个例子：
```
count:{1 TO 10]
```
    


### 用“^”增加一个术语

Lucene / Solr 根据找到的术语提供匹配文档的相关级别。要增加一个术语：^，请在搜索的术语末尾使用带有一个 boost 因子 (一个数字)的插入符号 ^。boost 因子越高，该术语的相关性越高。
      
  
增强功能使您可以通过增加文档的术语来控制文档的相关性。例如，如果您正在寻找 “jakarta apache”，您希望术语“jakarta”更相关，您可以通过在该术语后立即加上^ 符号和 boost 因子来提高它。例如，你可以输入：  
```
jakarta^4 apache
```

这将使与 jakarta 一词的文件显得更相关。您还可以像在示例中那样增加短语术语：  
```
"jakarta apache"^4 "Apache Lucene"
```

默认情况下，boost 因子为1。尽管 boost 因子必须为正值，但可小于1（例如，可能为0.2）。  

### 常量 Score 查询使用 “^ =” 

常量 score 查询是使用 &lt;query_clause&gt;^=&lt;score&gt; 创建的，它将整个子句设置为与该子句匹配的任何文档的指定分数。如果您只关心特定条款的匹配，而不希望其他相关因素，例如术语频率（该术语在该字段中出现的次数）或相反的文档频率（一项衡量整个指数中一个术语在一个领域的罕见程度的指标）。
      
  
例如：  
```
(description:blue OR color:blue)^=1.0 text:shoes
```


## 查询特定字段

在 Solr 中索引的数据是在 Solr Schema 中定义的字段中组织的。搜索可以利用字段来为查询增加精度。例如，您只能在特定字段（如标题字段）中搜索术语。
      
  
架构将一个字段定义为默认字段。如果您在查询中未指定字段，则 Solr 仅搜索默认字段。或者，您可以在查询中指定不同的字段或字段的组合。  
要指定一个字段，请输入字段名称，后跟一个冒号“：”，然后在该字段内搜索该术语。  
例如，假设索引包含两个字段，即标题和文本，并且该文本是默认字段。如果您想找到一个名为“The Right Way”，其中包含文本“don’t go this way”，您可以在您的搜索查询中包括以下任一项：  
```
title:"The Right Way" AND text:go
title:"Do it right" AND go
```

由于文本是默认字段，所以不需要字段指示符；因此上面的第二个查询省略了它。  
该字段仅对它直接位于其前面的术语有效，因此查询 title:Do it right 将在标题字段中仅找到 “Do”。它会在默认字段（在这里是文本字段）中找到 “it” 和 “right”。  

## 标准查询解析器支持的布尔运算符

布尔运算符允许您将布尔逻辑应用于查询，要求在字段中存在或不存在特定的术语或条件以匹配文档。下表总结了标准查询解析器支持的布尔运算符。  
<table class="">
    <colgroup>
        <col/>
            <col/>
                <col/>
    </colgroup>
    <thead>
        <tr>
            <th style="text-align: center;">布尔运算符</th>
            <th style="text-align: center;">替代符号</th>
            <th style="text-align: center;">描述</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <p style="text-align: center;">AND  
            </td>
            <td>
                <p style="text-align: center;"><code>&amp;&amp;</code>
                  
            </td>
            <td>
                要求布尔运算符两边的条件都存在以进行匹配。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">NOT  
            </td>
            <td>
                <p style="text-align: center;"><code>!</code>
                  
            </td>
            <td>
                要求下列术语不存在。  
            </td>
        </tr>
        <tr>
            <td>
                <p style="text-align: center;">OR  
            </td>
            <td>
                <p style="text-align: center;"><code>||</code>
                  
            </td>
            <td>
                要求两个术语 (或两个术语) 都存在于匹配项中。
                      
                  
            </td>
        </tr>
        <tr>
            <td/>
            <td>
                <p style="text-align: center;"><code>+</code>
                  
            </td>
            <td>
                要求提供以下术语。
                      
                  
            </td>
        </tr>
        <tr>
            <td/>
            <td>
                <p style="text-align: center;"><code>-</code>
                  
            </td>
            <td>
                禁止以下术语（即，不包含该术语的字段或文档上的匹配项）。该<code>-</code>运算符的功能类似布尔运算符<code>!</code>。由于它被 Google 等流行的搜索引擎所使用，因此一些用户社区可能会比较熟悉。  
            </td>
        </tr>
    </tbody>
</table>
布尔运算符允许通过逻辑运算符来组合术语。Lucene 支持 AND、“+”、OR、NOT 和“ - ”作为布尔运算符。
      
  
注意：使用 AND 或 NOT 等关键字指定布尔运算符时，关键字必须全部大写；标准查询解析器可以支持上表中列出的所有布尔运算符，DisMax 查询解析器仅支持<code>+</code>和<code>-</code>。  
OR 运算符是默认的连接运算符。这意味着，如果两个术语之间没有布尔运算符，则使用 OR 运算符。OR 运算符链接两个术语，如果文档中的任一术语存在，则查找匹配的文档。这等效于使用集的联合。符号 || 可以用来代替单词 OR。
      
  
要搜索包含 "jakarta apache" 或只是 "jakarta" 的文档，请使用以下查询：  
```
"jakarta apache" jakarta
```

或者：  
```
"jakarta apache" OR jakarta
```


### 布尔运算符“+”

该 + 符号（也称为“required”运算符）要求在至少一个文档中的某个字段中的 "+" 符号之后的术语存在，以便查询返回匹配。
      
  
例如，要搜索必须包含 “jakarta” 且可能包含 “lucene” 的文档，请使用以下查询：  
```
+jakarta lucene
```
标准查询解析器和DisMax查询解析器都支持此运算符。  

### 布尔运算符 AND（“&amp;&amp;”）

AND 运算符匹配单个文档的文本中任何地方存在两个词的文档。这相当于使用集合的交集。符号 &amp;&amp; 可以用来代替单词 AND。
      
  
要搜索包含 “jakarta apache” 和 “Apache Lucene” 的文档，请使用以下任一查询：  
```
"jakarta apache" AND "Apache Lucene"
"jakarta apache" &amp;&amp; "Apache Lucene"
```


### 布尔运算符NOT（“！”）

NOT 运算符排除在 NOT 之后包含该术语的文档。这相当于使用集合的差异。符号 ! 可以用来代替单词 NOT。
      
  
以下查询搜索包含短语 “jakarta apache” 但不包含短语 “Apache Lucene” 的文档：  
```
"jakarta apache" NOT "Apache Lucene"
"jakarta apache" ! "Apache Lucene"
```


### 布尔运算符“ - ”

该 - 符号或 “prohibit” 操作符排除包含 - 符号后的术语的文档。
      
  
例如，要搜索包含 “jakarta apache” 而不是 “Apache Lucene” 的文档，请使用以下查询：  
```
"jakarta apache" -"Apache Lucene"
```


### 逃离特殊字符

在查询中出现以下字符时，Solr 提供了特殊的含义：
      
  
```
+ - &amp;&amp; || ! ( ) { } [ ] ^ " ~ * ? : /
```

为了使 Solr 逐字地解释这些字符中的任何一个，而不是像特殊字符一样，在字符前面加一个反斜杠字符 \。例如，要搜索（1 + 1）：2 而不用 Solr 将加号和括号解释为用两个术语来表述子的特殊字符，请在每个字符前面加上反斜杠来转义字符：  
```
\(1\+1\)\:2
```


## 将术语分组以形成子查询

Lucene / Solr 支持使用括号将子句分组以形成子查询。如果要控制查询的布尔逻辑，这可能非常有用。
      
  
下面的查询搜索“jakarta”或“apache”和“website”：  
```
(jakarta OR apache) AND website
```

这增加了查询的精确度，要求使用术语“website”以及术语“jakarta”和“apache”。  

### 在一个字段中分组子句

要将两个或多个布尔运算符应用于搜索中的单个字段，请将括号中的布尔子句分组。例如，下面的查询搜索一个包含单词 “return” 和短语 “pink panther”的标题字段：
      
  
```
title:(+return +"pink panther")
```


## 查询中的注释

查询字符串支持 C 语言风格的注释。  
例如：  
```
"jakarta apache" /* this is a comment in the middle of a normal query string */ OR jakarta
```

评论可以嵌套。  

## Lucene 查询解析器和 Solr 标准查询解析器之间的区别


    - A * 可以用于指定一个或两个端点的一个不限范围查询：<ul><li>field:[* TO 100]：查找小于或等于100的所有字段值；
- field:[100 TO *]：查找大于或等于100的所有字段值；
- field:[* TO *]：将所有文档与该字段进行匹配
</li>
    <li>纯粹的负面查询（所有条款禁止）是允许的（仅作为 top-level 子句）：- inStock:false：找到 inStock 不为 false 的所有字段值；
- field:[* TO *]：找到所有没有字段值的文档
</li>
    <li>FunctionQuery 语法中的一个 hook。如果函数包含圆括号，则需要使用引号来封装该函数，如下面的第二个示例所示：
```
val:myfield
```
```
val:"recip(rord(myfield),1,2,3)"
```
    </li>
    <li>支持使用任何类型的查询解析器作为嵌套子句。
```
inStock:true OR {!dismax qf='name manu' v='ipod'}
```
    </li>
    <li>支持一个特殊的 filter(…​) 语法来指示某些查询子句应该被缓存在过滤器缓存中（作为一个常数分数布尔查询）。这允许子查询被缓存并在其他查询中重新使用。例如，inStock:true 将在下面的所有三个查询中被缓存和重新使用：
```
q=features:songs OR filter(inStock:true)
```
```
q=+manu:Apple +filter(inStock:true)
```
```
q=+manu:Apple &amp; fq=inStock:true
```
        这甚至可以用来缓存复杂过滤器查询的单个子句。在下面的第一个查询中，将3个项目添加到过滤器缓存（顶级 fq 和两个 filter(…​) 子句）中，在第二个查询中，将有2个缓存命中和一个新的缓存插入（用于新的顶级 fq）：  
```
q=features:songs &amp; fq=+filter(inStock:true) +filter(price:[* TO 100])
```
    </li>
    <li>
```
q=manu:Apple &amp; fq=-filter(inStock:true) -filter(price:[* TO 100])
```
    </li>
    <li>范围查询（“[a TO z]”），前缀查询（“a *”）和通配符查询（“a * b”）是常量计分（所有匹配的文档得到相同的 score）。评分因子 TF、IDF、指数提升和“coord”未被使用。对匹配项的数量没有限制（就像以前版本的 Lucene 一样）。</li>
    <li>常量分数查询是用 &lt;query_clause&gt;^=&lt;score&gt; 创建的，它将整个子句设置为匹配该子句的任何文档的指定分数：
```
q=(description:blue color:blue)^=1.0 title:blue^=5.0
```
    </li>
</ul>

### 指定日期和时间

基于日期的字段查询必须使用适当的日期格式。对于确切日期值的查询将需要引用或转义，因为是用于表示字段查询的解析器语法：  

    - createdate:1976-03-06T23\:59\:59.999Z
    - createdate:"1976-03-06T23:59:59.999Z"
    - createdate:[1976-03-06T23:59:59.999Z TO *]
    - createdate:[1995-12-31T23:59:59.999Z TO 2007-03-06T00:00:00Z]
    - timestamp:[* TO NOW]
    - pubdate:[NOW-1YEAR/DAY TO NOW/DAY+1DAY]
    - createdate:[1976-03-06T23:59:59.999Z TO 1976-03-06T23:59:59.999Z+1YEAR]
    - createdate:[1976-03-06T23:59:59.999Z/YEAR TO 1976-03-06T23:59:59.999Z]

