## 如何使用DocValues 
<div class="content-intro view-box ">DocValues 是一种在内部记录字段值的方法，与传统的索引相比，对于某些用途（如排序和 faceting）更高效。  
  

## 为什么使用 DocValues？<a href="http://lucene.apache.org/solr/guide/7_0/docvalues.html#why-docvalues"/>

Solr 建立索引的标准方式是倒排索引。这种风格生成了索引中所有文档中的术语列表，每个术语旁边是该术语出现的文档列表（以及该文档中出现的术语的次数）。这使得搜索速度非常快 - 因为用户按照术语进行搜索，具有 term-to-document 值的现成列表使查询过程更快。  
  
对于我们现在通常与搜索关联的其他功能，如排序，faceting 和突出显示，此方法效率不高。例如，faceting 引擎必须查看每个文档中出现的每个术语、这些术语将组成结果集并提取文档 ID 以生成 faceting 列表。在 Solr 中，这被保存在内存中，并且可能加载缓慢（取决于文档数量、术语等）。  
在 Lucene 4.0 中，引入了一种新的方法。DocValue 字段现在是面向列的字段，document-to-value 映射在索引时生成。这种方法有望减轻 fieldCache 的一些内存需求，并且可以更快地查找 faceting，排序和分组。  

## 启用 DocValues<a href="http://lucene.apache.org/solr/guide/7_0/docvalues.html#enabling-docvalues"/>

要使用 docValues，只需要为将要使用的字段启用它。与所有架构设计一样，您需要定义一个字段类型，然后使用 docValues 启用该类型的字段。所有这些行动都是在 schema.xml 中完成。  
为 docValues 启用一个字段只需要添加 docValues="true" 到字段（或字段类型）的定义，就像来自这个例子中 Solr 的 sample_techproducts_configs 配置集合 schema.xml 一样：  
```
&lt;field name="manu_exact" type="string" indexed="false" stored="false" docValues="true" /&gt;
```
如果您已经将索引数据编入了 Solr 索引中，则需要在更改 schema.xml 中的字段定义后为您的内容重新编制索引，以便成功使用 docValues。  
DocValues 只适用于特定的字段类型。选择的类型决定了将使用的底层 Lucene docValue 类型。可用的 Solr 字段类型为:  

    - StrField 和 UUIDField：如果该字段是单值（即多值为 false），则 Lucene 将使用该 SORTED 类型。如果该字段是多值的，Lucene 将使用该 SORTED_SET 类型。
    - BoolField：如果该字段是单值（即多值为 false），则 Lucene 将使用该 SORTED 类型。如果该字段是多值的，Lucene 将使用该 SORTED_BINARY 类型。
    - 任何 *PointField 数字或日期字段、EnumFieldType 和 CurrencyFieldType：如果该字段是单值（即多值为 false），则 Lucene 将使用该 NUMERIC 类型。如果该字段是多值的，Lucene 将使用该 SORTED_NUMERIC 类型。
    - 任何不建议使用的 Trie* 数字或日期字段、EnumField 以及 CurrencyField：如果该字段是单值（即多值为 false），则 Lucene 将使用该 NUMERIC 类型。如果该字段是多值的，Lucene 将使用该 SORTED_SET 类型。

这些 Lucene 类型与如何对值进行排序和存储相关。  
还有一个额外的配置选项可用，即修改 docValuesFormat 字段类型使用的选项。默认实现是将一些东西加载到内存中，并保留在磁盘上。但是，在某些情况下，您可以选择指定一个替代的 DocValuesFormat 实现。例如，您可以选择通过指定 docValuesFormat="Memory" 字段类型来将所有内容都保存在内存中：  
```
&lt;fieldType name="string_in_mem_dv" class="solr.StrField" docValues="true" docValuesFormat="Memory" /&gt;
```
请注意，该 docValuesFormat 选项可能会在将来的版本中更改。  
Lucene 索引向后兼容只支持默认编解码器。如果您选择在 schema.xml 中自定义 docValuesFormat，则升级到未来版本的 Solr 可能要求您切换回默认的编码解码器并优化索引, 以便在升级之前将其重写为默认编码解码器, 或重新构建整个升级后从头开始索引。  

## 使用 DocValues<a href="http://lucene.apache.org/solr/guide/7_0/docvalues.html#using-docvalues"/>

### 排序，Faceting 和功能

### <a href="http://lucene.apache.org/solr/guide/7_0/docvalues.html#sorting-faceting-functions"/>

如果 docValues="true"，则在字段用于排序、faceting 或函数查询时, docValues 将自动被使用。  

### 在搜索过程中检索 DocValues
在搜索查询期间检索的字段值通常从存储的值中返回。但是，当所有字段（或模式匹配 globs）被指定返回（例如“fl = *”）用于搜索查询时，non-stored docValues 字段也将与其他存储字段一起返回，取决于每个字段 useDocValuesAsStored 参数的有效值。对于架构版本&gt; = 1.6，隐式默认值为useDocValuesAsStored="true"。有关更多详细信息，请参阅字段类型定义和属性和定义字段。  
当 useDocValuesAsStored="false" 时，non-stored DocValues 字段仍然可以通过 fl 参数中的名字明确请求，但不会匹配 glob 模式（"*"）。请注意，在查询时返回 DocValues 以及“常规”存储字段具有性能影响，因为存储的字段可能并不是因为 DocValue 是列式的，因此可能会为每个返回的文档产生额外的成本检索。另请注意，在从 DocValues 返回 non-stored 字段时，多值字段的值将按排序顺序（而不是插入顺序）返回。如果您需要在原始插入顺序中返回多值字段，请将您的多值字段存储（如此更改需要重新编制索引）。  
在查询返回的情况下，只有 docValues 字段的性能可能会提高，因为返回的存储字段需要磁盘读取和解压缩，而返回 fl 列表中的 docValues 字段只需要访问内存。  
从 docValues 表单中检索字段（使用 / export 处理程序、流式表达式或在 fl 参数中请求字段时），则必须理解常规存储字段和 docValues 字段之间的两个重要区别：  
1 <li>订单不被保留。为了简单地检索存储的字段，插入顺序是返回顺序。对于 docValues，这是排序顺序。</li>2 <li>对于使用 SORTED_SET 的字段类型，将多个相同的条目合并为一个值。因此，如果插入值 4,5,2,4,1，则回报将是 1，2，4，5。</li>
