# Schema的元素

本节介绍在前面几节中未提及的 schema.xml 的其他几个重要的模式元素。

## uniqueKey主键

uniqueKey元素用来表示一个文档的唯一标识域，Solr使用此域决定增量导入时是否重复导入,如果uniqueKey相同,则不再重复导入,更新索引时,也可使用uniqueKey来确定唯一文档对该文档更新。尽管uniqueKey不是必须，但在程序设计过程中还是建议使用。

```xml
<field name="id" type="uuid" indexed="true" stored="true" required="true"/>
<uniqueKey>id</uniqueKey>
```

>注:Schema默认值和 copyFields 不能用于uniqueKey 字段。

在 fieldType 中 uniqueKey 不得进行分析。您可以使用 UUIDUpdateProcessorFactory 自动生成具有 uniqueKey的值。

## copyField复制字段

copyField，可以将多个Field复制到一个Field中，以便进行统一的检索. 比如，输入关键字要搜索title标题和内容content这两个字段时，要用到复制copyField。

目标字段定义:
`<filed name="text" type="text_general" stored="false" indexed="true" multiValued="true">`

*注意必须是multivalued="true"*

将下面定义字段复制到目标字段中:

```xml
<copyField source="title" dest="text" />
<copyField source="content" dest="text" maxChars="2000" />
<copyField source="features" dest="text" />
```

如果在搜索时，搜索text域，solr会分别从以上title、content、features这字段中搜索。

maxChars 参数是一个 int 参数，用于在构造添加到目标字段的值时，为要从源值复制的字符数建立一个上限。此限制对于要从源字段复制某些数据的情况非常有用，而且还可以控制索引文件的大​​小。

copyField 的源和目标都可以包含前导或尾随星号，这将匹配任何内容。例如，下面的行将与通配符模式 * _t 匹配的所有传入字段的内容复制到文本字段中：

```xml
<copyField source="*_t" dest="text" maxChars="25000" />
```

>只有当 source 参数也包含一个参数时，该 copyField 命令才可以在 dest 参数中使用通配符（*）。copyField 使用源字段中匹配的 glob dest 作为源内容复制到的字段名称。

复制是在流源级别完成的，并且不复制到另一个副本中。这意味着复制字段不能被链接，即不能从 here 复制到 there 然后从 there 复制到 elsewhere。但是，可以将相同的源字段复制到多个目标字段：

```xml
<copyField source="here" dest="there"/>
<copyField source="here" dest="elsewhere"/>
```

## dynamicField动态字段

dynamicField允许 Solr 对您在架构中未明确定义的字段进行索引。

动态字段的name属性是带有通配符的,在作为索引文档时，与任何明确定义的字段都不匹配的字段可以与动态字段匹配。

例如，假设您的模式中包含一个具有 *_i 名称的动态字段。如果您尝试使用 cost_i 字段对文档进行索引，但架构中没有定义明确的 cost_i 字段，则该 cost_i 字段将具有*_i 定义的字段类型和分析。
像常规字段一样，动态字段具有名称、字段类型和选项。

```xml
<dynamicField name="*_i" type="int" indexed="true"  stored="true"/>
```

## Similarity

Similarity是一个Lucene类，用于在搜索中对文档进行评分。

每个集合都有一个“全局”相似性，默认情况下，Solr 使用一个隐式 SchemaSimilarityFactory，它允许将单个字段类型配置一个“每类型”特定的相似性，并隐式使用 BM25Similarity 对任何字段类型没有明确的相似性。  
可以通过在 schema.xml 中的顶级 &lt;similarity/&gt; 元素(在任何单个字段类型之外) 来重写此默认行为。这个相似性声明可以直接引用具有无参数构造函数的类的名称，如下例中显示 BM25Similarity：

`<similarity class="solr.BM25SimilarityFactory"/>`

或者引用 SimilarityFactory 实现，它可能采用可选的初始化参数:

```xml
<similarity class="solr.DFRSimilarityFactory">
  <str name="basicModel">P</str>
  <str name="afterEffect">L</str>
  <str name="normalization">H2</str>
  <float name="c">7</float>
</similarity>
```

在大多数情况下， 如果您的 schema.xml 还包括字段类型特定的 `<similarity/>` 声明，则指定全局级别相似性会导致错误。一个重要的例外是，您可以明确地声明一个SchemaSimilarityFactory，并指定默认行为对于所有不使用字段类型名称 (由 defaultSimFromFieldType 指定) 声明显式相似性的字段类型。具有特定相似性的配置：

```xml
<similarity class="solr.SchemaSimilarityFactory">
  <str name="defaultSimFromFieldType">text_dfr</str>
</similarity>
<fieldType name="text_dfr" class="solr.TextField">
  <analyzer ... />
  <similarity class="solr.DFRSimilarityFactory">
    <str name="basicModel">I(F)</str>
    <str name="afterEffect">B</str>
    <str name="normalization">H3</str>
    <float name="mu">900</float>
  </similarity>
</fieldType>
<fieldType name="text_ib" class="solr.TextField">
  <analyzer ... />
  <similarity class="solr.IBSimilarityFactory">
    <str name="distribution">SPL</str>
    <str name="lambda">DF</str>
    <str name="normalization">H2</str>
  </similarity>
</fieldType>
<fieldType name="text_other" class="solr.TextField">
  <analyzer ... />
</fieldType>
```

在上面的示例中，IBSimilarityFactory（使用基于信息的模型）将用于 text_ib 类型的任何字段，而 DFRSimilarityFactory（随机的分歧）将用于 text_dfr 类型的任何字段，以及使用类型的任何字段，该类型没有明确指定 `<similarity/>`。

如果 SchemaSimilarityFactory 是通过配置 defaultSimFromFieldType 显式声明的，则 BM25Similarity 将隐式用作默认值。

除了此页面上提到的各个工厂，还有其他几种类似的实现，如：SweetSpotSimilarityFactory、ClassicSimilarityFactory 等等。有关详细信息，请参见 Solr Javadocs 的相似工厂。
