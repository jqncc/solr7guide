## Solr函数查询的使用 
<div class="content-intro view-box ">Solr 函数查询使您能够使用一个或多个数字字段的实际值生成相关性分数。  
  
Solr 函数查询由 DisMax、Extended DisMax 和标准查询解析器支持。  
Solr 函数查询使用函数。函数可以是常量（数字或字符串文字）、字段、另一个函数或参数替换参数。您可以使用这些函数来修改用户结果的排名。这些可用于根据用户的位置或其他计算来更改结果的排序。  
## 使用函数查询
函数必须表达为函数调用（例如，sum(a,b) 而不是简单地 a+b）。  
在 Solr 查询中有几种使用函数查询的方法：  
- 通过一个明确的 QParser，期望函数参数，例如 func 或 frange：
```
q={!func}div(popularity,price)&amp;fq={!frange l=1000}customer_ratings
```
- 在一个 Sort 表达式中。例如：
```
sort=div(popularity,price) desc, score desc
```
- 将函数的结果作为伪字段（pseudo-fields）添加到查询结果中的文档。例如，对于：
```
&amp;fl=sum(x, y),id,a,b,c,score
```
输出将是：  
```
...
&lt;str name="id"&gt;foo&lt;/str&gt;
&lt;float name="sum(x,y)"&gt;40&lt;/float&gt;
&lt;float name="score"&gt;0.343&lt;/float&gt;
...
```
- 用于显式指定函数的参数，如 EDisMax 查询解析器的 boost 参数或 DisMax 查询解析器 bf（boost 函数）参数。（请注意，bf 参数实际上是用空格分隔的函数查询列表，每个函数都有一个可选的提升函数，确保在使用 bf 时，一定要消除单个函数查询中的任何内部空白）。例如：
```
q=dismax&amp;bf="ord(popularity)^0.5 recip(rord(price),1,1000,1000)^0.3"
```
- 在 lucene QParser 中用 _val_ 关键字在内联中引入一个函数查询。例如：
```
q=_val_:mynumericfield _val_:"recip(rord(myfield),1,2,3)"
```
建议只使用具有快速随机访问功能的函数。  
