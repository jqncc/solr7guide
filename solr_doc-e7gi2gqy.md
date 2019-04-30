## Solr查询中的本地参数 
<div class="content-intro view-box ">
## 本地参数
本地参数是 Solr 请求中特定于查询参数的参数。  
  
本地参数提供了将元数据添加到某些参数类型（如查询字符串）的方法。（在 Solr 文档中，本地参数有时被称为 LocalParams。）  
本地参数被指定为参数的前缀。以下面的查询参数为例：  
```
q=solr rocks
```
我们可以使用本地参数对此查询字符串进行前缀，为标准查询解析器提供更多的信息。例如，我们可以将默认的操作符类型更改为“AND”，将默认的字段更改为“title”：  
```
q={!q.op=AND df=title}solr rocks
```
这些本地参数会在默认搜索“title”字段的同时将查询更改为“solr”和“rocks”。  
## 本地参数的基本语法<a href="http://lucene.apache.org/solr/guide/7_0/local-parameters-in-queries.html#basic-syntax-of-local-parameters"/>
要指定一个本地参数，请在要修改的参数前插入以下内容：  
- 首先：{!
- 插入由空格分隔的任意数量的 key=value
- 以 } 结束并立即跟随查询参数
每个参数只能指定一个本地参数前缀。key-value 对中的值可以通过单引号或双引号引用，并且在带引号的字符串中使用反斜杠转义。  
## 查询类型缩写<a href="http://lucene.apache.org/solr/guide/7_0/local-parameters-in-queries.html#query-type-short-form"/>
如果一个本地参数值没有名字出现，它会被赋予一个隐含的名字“type”。这允许在解析查询字符串时使用查询解析器类型的简短表示。从而：  
```
q={!dismax qf=myfield}solr rocks
```
相当于：  
```
q={!type=dismax qf=myfield}solr rocks
```
如果没有指定“type”（显式或隐式），则默认使用 lucene 分析器。从而：  
```
fq={!df=summary}solr rocks
```
等同于：  
```
fq={!type=lucene df=summary}solr rocks
```
## 用v 键指定参数值<a href="http://lucene.apache.org/solr/guide/7_0/local-parameters-in-queries.html#specifying-the-parameter-value-with-the-v-key"/>
本地参数中的 v 的特殊键是指定该参数的值的替代方法：  
```
q={!dismax qf=myfield}solr rocks
```
相当于：  
```
q={!type=dismax qf=myfield v='solr rocks'}
```
## 参数取消引用<a href="http://lucene.apache.org/solr/guide/7_0/local-parameters-in-queries.html#parameter-dereferencing"/>
通过参数取消引用或间接引用，可以使用另一个参数的值，而不是直接指定它的值。这可以用来简化查询，将用户输入从查询参数中分离出来，或者将前端 GUI 参数从 solrconfig. xml 中的默认设置中分离出来。  
```
q={!dismax qf=myfield}solr rocks
```
等同于：  
```
q={!type=dismax qf=myfield v=$qq}&amp;qq=solr rocks
```
