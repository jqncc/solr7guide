# 日期字段类型

## Solr 日期格式

Solr的日期字段（DatePointField，DateRangeField 和已过时的 ~~TrieDateField~~）代表“dates”，如时间精确到毫秒的点。使用的格式是XML Schema规范中dateTime的规范表示的受限形式- ISO-8601的受限子集。对于熟悉Java 8的人，Solr使用DateTimeFormatter.ISO_INSTANT进行格式化，并使用“leniency”进行解析。

`YYYY-MM-DDThh:mm:ssZ`

其中：

- YYYY 年份。
- MM 月份。
- DD 月份的某一天。
- hh 24小时制中一天内的时间。
- mm 分钟。
- ss 秒。
- Z 表示UTC时区。

请注意，不能指定时区。日期的字符串表示形式始终以协调世界时 (UTC) 表示。下面是一个示例值：

`1972-05-20T17:33:18Z`

如果您愿意，您可以选择包含小数的秒数，但任何超出毫秒的精度都将被忽略。下面是 sub-seconds 的示例值：

- 1972-05-20T17:33:18.772Z
- 1972-05-20T17:33:18.77Z
- 1972-05-20T17:33:18.7Z

0000年前的日期前加"-"号，而9999年之后日期以“+”开头。年0000被认为是公元前1年；没有公元 0 年或公元前 0 年的表达方法。

>可能需要查询转义：
>如您所见，日期格式包括分隔小时、分钟和秒的冒号字符。因为冒号是 Solr 最常见的查询解析器的特殊字符，所以有时需要转义，具体取决于您正在尝试执行的操作。
>这通常是一个无效查询:datefield: 1972-05-20T17:33:18.772Z
>以下这些是有效的查询:

datefield: 1972-05-20T17 \:33 \: 18.772Z  
datefield: "1972-05-20T17:33:18.772Z"  
datefield: [1972-05-20T17:33:18.772Z TO *]  

### 日期范围格式

Solr DateRangeField 既支持上述时间点语法（也具有下面描述的日期数学）以及更多的表达日期范围。一类示例是截断的日期，它表示整个日期跨度到所指示的精度。另一个类使用范围语法（[ TO ]）。下面是一些示例：

- 2000-11：2000年11月的整个月
- 2000-11T13：同样，但在一天中的一个小时（1300至1400前，即下午1点至2点）。
- -0009：公元前10年，年份中的 0 是公元 0，也被认为是公元前1年。
- [2000-11-01 TO 2014-12-01]：指定的日期范围在一天的分辨率。
- [2014 TO 2014-12-01]：从 2014 年开始到 12 月的第一天结束。
- [* TO 2014-12-01]：从最早的可代表时间到2014年12月12日的一天结束。

限制：范围语法不支持嵌入日期数学。如果您指定了 DatePointField 支持的日期实例，并将日期数学截断，比如 NOW/DAY，那么您仍然可以得到当天的第一个毫秒，而不是整天的范围。独占范围（使用{＆}）在查询中工作，但不适用于索引范围。

## 日期数学Date Math

Solr 的日期字段类型也支持日期数学表达式，这使得相对于固定时刻的时间创建变得容易，包括使用“NOW”的特殊值来表示的当前时间。

### 日期数学表达式

日期数学表达式包括在指定的单位中添加一定量的时间，或者以指定单位四舍五入当前时间。表达式可以链接并从左向右进行评估。

例如：以下代表了两个月后的时间点：

`NOW+2MONTHS`

这是一天前：
`NOW-1DAY`

斜线用于表示舍入。这代表当前小时的开始：
`NOW/HOUR`


下面的例子计算（精确到毫秒）时间为六个月和三天后的时间点，然后将该时间点回溯到当天的开始时间：

`NOW+6MONTHS+3DAYS/DAY`

请注意，虽然日期数学是相对于 NOW 是最常用的，但它也可以应用于任何固定时间点：

`1972-05-20T17:33:18.772Z+6MONTHS+3DAYS/DAY`


### 影响日期数学的请求参数


#### NOW

该 NOW 参数由 Solr 在内部使用，以确保在分布式请求中的多个节点上分析一致的日期数学表达式。但是可以指定 Solr 使用任意时刻（过去或将来）的任意时刻覆盖“ NOW” 的特殊值会影响日期数学表达式的所有情况。

它必须被指定为自纪元以来的（长值）毫秒。
示例：  
```
q=solr&amp;fq=start_date:[* TO NOW]&amp;NOW=1384387200000
```

#### TZ

默认情况下，所有日期数学表达式都是相对于 UTC TimeZone 计算的，但是 TZ 可以指定该参数来覆盖此行为，方法是强制所有基于日期的加法和舍入相对于指定的时区。

例如，下面的请求将使用范围面来面向当前月份，“每天”相对 UTC：
```
http://localhost:8983/solr/my_collection/select?q=*:*&amp;facet.range=my_date_field&amp;facet=true&amp;facet.range.start=NOW/MONTH&amp;facet.range.end=NOW/MONTH%2B1MONTH&amp;facet.range.gap=%2B1DAY
```

```
<int name="2013-11-01T00:00:00Z">0</int>
<int name="2013-11-02T00:00:00Z">0</int>
<int name="2013-11-03T00:00:00Z">0</int>
<int name="2013-11-04T00:00:00Z">0</int>
<int name="2013-11-05T00:00:00Z">0</int>
<int name="2013-11-06T00:00:00Z">0</int>
<int name="2013-11-07T00:00:00Z">0</int>
...</code>
    </pre>
    在本例中，“days” 将根据指定的时区进行计算 - 包括任何适用的夏令时间调整：  
<pre lang="javascript" style="max-width: 100%;"><code class="javascript">http://localhost:8983/solr/my_collection/select?q=*:*&amp;facet.range=my_date_field&amp;facet=true&amp;facet.range.start=NOW/MONTH&amp;facet.range.end=NOW/MONTH%2B1MONTH&amp;facet.range.gap=%2B1DAY&amp;TZ=America/Los_Angeles
```
```
<int name="2013-11-01T07:00:00Z">0</int>
<int name="2013-11-02T07:00:00Z">0</int>
<int name="2013-11-03T07:00:00Z">0</int>
<int name="2013-11-04T08:00:00Z">0</int>
<int name="2013-11-05T08:00:00Z">0</int>
<int name="2013-11-06T08:00:00Z"></int>0</int>
<int name="2013-11-07T08:00:00Z">0</int>
...
```

## 更多 DateRangeField 详细信息

DateRangeField 几乎是 DatePointField 所用地方的替代品。唯一的区别是，Solr 的 XML 或 SolrJ 响应格式将存储的数据作为一个字符串而不是一个日期。该字段的基础索引数据会稍大一些。与时间单位对齐的查询应该比 TrieDateField 快，尤其是在 UTC 的情况下。  

DateRangeField，顾名思义，其主要观点是允许索引日期范围。要做到这一点，只需提供上面显示的格式的字符串。它还支持在索引数据和查询范围之间指定3个不同的关系谓词：  

- Intersects （默认）
- Contains
- Within

你可以通过查询使用 oplocal-params 参数来指定谓词，如下所示：

`fq={!field f=dateRange op=Contains}[2013 TO 2018]`

与大多数本地参数不同，op 实际上不是由任何查询解析器（field）定义的，它是由字段类型定义的，在本例中为 DateRangeField。在上面的示例中，它将查找包含 (或等于) 范围2013到2018的索引范围的文档。文档中的多值重叠索引范围有效地合并。
有关 DateRangeField 示例用例，请参阅 Solr 的社区 wiki。
