## 如何对Solr查询进行重新排序 
<div class="content-intro view-box ">
## <span style="font-family: inherit;">查询</span>重新排序
Solr 查询重新排序允许您运行一个简单的查询（A）来匹配文档，然后使用来自更复杂的查询（B）的分数重新排列前 N 个文档。  
  
由于从查询 B 中获得的更高的排名仅适用于前 N 个文档，因此它对性能的影响将较小，仅使用复杂的查询 B本身。折衷的是，在重新排序阶段，使用简单查询 A 得分非常低的文档可能不被考虑，即使它们使用查询 B 的得分非常高。  
## 指定排序查询
排名查询可以使用 rq 请求参数指定。该 rq 参数必须指定一个查询字符串，在解析时产生一个 RankQuery。  
  
Solr 发行版中目前包含三个等级查询。您也可以配置您编写的自定义 QParserPlugin，但大多数用户可以使用 Solr 提供的解析器，如下表所示：  
<table class=""><colgroup><col/><col/></colgroup><thead><tr><th style="text-align: center;">解析器</th><th style="text-align: center;">QParserPlugin 类</th></tr></thead><tbody><tr><td><p style="text-align: center; ">RerRank  
</td><td><p style="text-align: center;">ReRankQParserPlugin  
</td></tr><tr><td><p style="text-align: center;">XPORT  
</td><td><p style="text-align: center;">ExportQParserPlugin  
</td></tr><tr><td><p style="text-align: center;">LTR  
</td><td><p style="text-align: center;">LTRQParserPlugin  
</td></tr></tbody></table>
### ReRank 查询解析器
该 rerank 解析器包装由本地参数指定的查询，以表示有多少文件应被重新排序以及如何计算最终分数的附加参数：  
  
**reRankQuery**
复杂排序查询的查询字符串 - 在大多数情况下，变量将用于引用另一个请求参数。该参数是必需的。  
**reRankDocs**
来自原始查询的前 N 个文档的数量应该重新排列。这个数字将被视为最低限度，并可能会自动增加内部，以便为满足查询 (即, 启动 + 行) 排列足够的文档。默认是<code>200</code>。  
**reRankWeight**
在将得分添加到原始得分之前，将应用于来自 reRankQuery 的每个最高匹配文档的得分的乘法因子。默认是<code>2.0</code>。  

在下面的示例中，匹配查询“greetings”的前1000个文档将使用查询“(hi hello hey hiya)”重新排序。这1000个文件中的每一个得到的分数将是“（hello hey hiya）”得分的3倍，再加上原始“greetings”查询的得分：  
```
q=greetings&amp;rq={!rerank reRankQuery=$rqq reRankDocs=1000 reRankWeight=3}&amp;rqq=(hi+hello+hey+hiya)
```
如果文档与原始查询匹配，但与重新排列的查询不匹配，则文档的原始分数将保留。  
### LTR 查询解析器
ltr 代表学习排名，请参阅学习排名获得更详细的信息。  
  
## 将排序查询与其他 Solr 特征相结合
一般而言，rq 参数和重新排序功能可与其他 Solr 功能配合使用。例如，可以将它与折叠解析器一起使用，以便在折叠后重新排列组头。它还保留了由高程组件提升的文档的顺序。它甚至有自己的自定义解释，所以您可以看到如何在查看调试信息时得到重新评分的。  
