# Solr如何定义字段 

字段是在 schema.xml 的字段元素中定义的。一旦你设置了字段类型，那么定义 Solr 字段本身很简单了。

## 示例-字段定义

以下示例定义了一个类型名为 float 并且默认值为 0.0 的名为 price 的字段；indexed 和 stored 特性明确地设置为 true，而在 float 字段类型上指定的任何其他属性都将被继承。

```xml
<field name="price" type="float" default="0.0" indexed="true" stored="true"/>
```

## 字段属性

字段定义可以具有以下属性：

**name**

该字段的名称。字段名称只能由字母数字或下划线字符组成，不能以数字开头。目前这并不是严格执行的，但其他字段名称将不具备所有组件的第一类支持，并且不保证向后的兼容性。带有前导和后缀下划线的名称（例如，_version_）被保留。每个字段都必须有一个name。

**type**

该fieldType字段的名称。这将name在fieldType定义的name属性中找到。每个字段都必须有一个type。  

**default**

将自动添加到在索引时该字段中没有值的任何文档的默认值。如果这个属性没有指定，那么没有默认值。  

## 可选的字段类型重写属性

字段可以具有许多与字段类型相同的属性。下表中的属性在单个字段中指定，将重写在字段的 fieldType 上指定的该属性的任何显式值，或者由基础 fieldType 实现所提供的任何隐式默认属性值。下表从字段类型定义和属性转载，其中有更多详细信息：

属性|描述|取值|隐含默认值
|---|--|--|--
indexed|如果为 true，则可以在查询中使用该字段的值来检索匹配的文档。|true 或者 false|true  
stored|如果为 true，则字段的实际值可以通过查询来检索。|true 或者 false|true
docValues|如果为 true，则该字段的值将被放入一个面向列的 DocValues 结构中。|true 或者 false|false sortMissingFirst / sortMissingLast|排序字段不存在时控制文档的位置。|true 或者 false|false
multiValued|如果为 true，则表示单个文档可能包含此字段类型的多个值。|true 或者 false|false
omitNorms|如果为 true，则省略与该字段关联的规范（这将禁用该字段的长度规范化，并保存一些内存）。对于所有基元 (non-analyzed) 字段类型（如 int、float、data、bool 和 string）的默认值均为true。只有全文字段或字段需要规范。|true 或者 false|*  
omitTermFreqAndPositions| 如果为 true，则省略该字段过帐的术语频率、位置和有效载荷。这可以提高不需要这些信息的字段的性能。这也减少了索引所需的存储空间。依赖于使用此选项在字段上发布的位置的查询将悄然无法找到文档。对于不是文本字段的所有字段类型，此属性默认为 true。|true 或者 false|*  
omitPositions|类似于omitTermFreqAndPositions但保留了词频信息|true 或者 false|*  
termVectors termPositions termOffsets termPayloads|这些选项指示 Solr 维护每个文档的全部向量矢量，可选地包括这些向量中每个词条出现的位置，偏移和有效载荷信息。这些可以用来加速突出显示和其他辅助功能，但在索引大小方面会带来相当大的成本。对于 Solr 的典型用途，它们不是必需的。|true 或者 false|false
required|指示 Solr 拒绝任何尝试添加一个文件，该文件没有这个字段的值。该属性默认为 false。|true 或者 false|false
useDocValuesAsStored|如果该字段已docValues启用，则将其设置为true将允许stored=false在fl参数中匹配“*”时，将该字段作为存储字段返回（即使有）。|true 或者 false|true  
large|如果实际值&lt;512KB，则大字段总是被延迟加载，并且只占用文档高速缓存中的空间。这个选项需要stored="true"和multiValued="false"。它的目的是为了可能有非常大的值，以便他们不被缓存在内存中的字段。|true 或者 false|false  
