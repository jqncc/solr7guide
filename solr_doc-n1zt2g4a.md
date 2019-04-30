## Solr如何收集碎片 
<div class="content-intro view-box ">最高级别的 schema.xml 结构如下。  
  
这个例子不是真正的 XML，但它为您提供了文件结构的概念。  
```
&lt;schema&gt;
  &lt;types&gt;
  &lt;fields&gt;
  &lt;uniqueKey&gt;
  &lt;copyField&gt;
&lt;/schema&gt;
```
显然，大部分的兴奋是在 types 和 fields 中，其中字段类型和实际字段定义的存在。  
  
这些由 copyFields 补充。  
必须始终定义 uniqueKey。  
类型和字段是可选的标记：请注意， "types" 和 "fields" 部分是可选的，这意味着您可以在顶层上自由搭配<code>field</code>、<code>dynamicField</code>、<code>copyField</code>和<code>fieldType</code>定义。这允许在架构中对相关标记进行更合理的分组。  
  

## 选择适当的数值类型<a href="http://lucene.apache.org/solr/guide/7_0/putting-the-pieces-together.html#choosing-appropriate-numeric-types"/>

对于一般的数字需求，可以考虑使用 IntPointField、LongPointField、FloatPointField 或 DoublePointField 类之一，具体取决于所期望的特定值。这些基于“维度点”的数字类使用特殊编码的数据结构来支持高效的范围查询，而不管所用范围的大小。根据需要在这些字段上启用 DocValues 进行排序或刻面。  
  
有些 Solr 功能可能还不能使用 "维度点"，在这种情况下，您可能需要考虑等效的 TrieIntField、TrieLongField、TrieFloatField 和 TrieDoubleField 类。这些字段类型已被弃用，并可能在未来的主要 Solr 版本中被删除，但仍可在必要时使用。配置 precisionStep="0" 如果希望最小化索引大小，但是如果您希望用户对数值类型进行频繁的范围查询，请使用默认的 precisionStep（不指定它）或将其指定为 precisionStep="8"（默认值）。这提高了范围查询的速度，但是增加了索引大小。  

## 使用文本
正确处理文本将使用户感到满意，为他们提供最佳的文本搜索结果。  
一种方法是使用文本字段作为关键字搜索的全部功能。大多数用户对他们的搜索并不熟悉，最常见的搜索可能是一个简单的关键字搜索。您可以使用 copyField 各种字段，并将它们全部汇入到单个文本字段中以进行关键字搜索。  
在包含 Solr 的 "techproducts" 示例的 Schema.xml 文件中，copyField 声明用于转储 cat、name、manu、features，和 includes 的内容成为单个字段：text。此外，最好将 ID 复制到 text 中，以防用户希望通过将其产品编号传递给关键字搜索来搜索特定产品。  
另一种方法是使用 copyField 以不同的方式使用相同的字段。假设您有一个作者列表的字段，如下所示：  
```
Schildt, Herbert; Wolpert, Lewis; Davies, P.
```
对于作者搜索，您可以标记字段，转换为小写，并删除标点符号：  
```
schildt / herbert / wolpert / lewis / davies / p
```
对于排序，只需使用 untokenized 字段，将其转换为小写字母，并删除标点符号：  
```
schildt herbert wolpert lewis davies p
```
最后，为了 faceting，仅通过 StrField 使用主作者：  
```
Schildt, Herbert
```
