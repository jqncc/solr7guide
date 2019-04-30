## Solr无架构模式 
无架构模式是一组 Solr 功能，它们一起使用时，用户只需简单地对示例数据进行索引就可以快速构建有效的架构，而无需手动编辑模式。  
  
这些 Solr 功能都是通过 solrconfig.xml 方式控制的：  
1 <li>托管架构：在运行时通过 Solr API 进行架构修改，这需要使用支持这些更改的 schemaFactory。有关更多详细信息，请参阅 SolrConfig 中的 "架构工厂定义" 部分。</li>2 <li>字段值类猜测：以前看不到的字段是通过一组级联的价值分析器来运行的，它们猜测 Java 类的字段值 - 布尔型、整数型、长整型、浮点型、双精度型和日期型的解析器当前可用。</li>3 <li>基于字段值类（es）的自动模式字段添加：基于字段值 Java 类（字段值映射到架构字段类型），将先前看不到的字段添加到模式中。 - 请参阅 Solr 字段类型。</li>
## 使用无架构示例<a href="http://lucene.apache.org/solr/guide/7_0/schemaless-mode.html#using-the-schemaless-example"/>

无架构模式的三个特性是在 Solr 分配的 _default 配置集中预先配置的。要使用这些配置启动 Solr 的示例实例，请运行以下命令：  
```
bin/solr start -e schemaless
```
这将启动一个 Solr 的服务器，并自动创建一个集合（名为 “gettingstarted 只包含三个初始架构字段”）：id、_version_ 和 _text_。  
你可以使用 /schema/fields Schema API 来确认：curl http://localhost:8983/solr/gettingstarted/schema/fields 输出：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":1},
  "fields":[{
      "name":"_text_",
      "type":"text_general",
      "multiValued":true,
      "indexed":true,
      "stored":false},
    {
      "name":"_version_",
      "type":"long",
      "indexed":true,
      "stored":true},
    {
      "name":"id",
      "type":"string",
      "multiValued":false,
      "indexed":true,
      "required":true,
      "stored":true,
      "uniqueKey":true}]}
```

## 配置无看过模式<a href="http://lucene.apache.org/solr/guide/7_0/schemaless-mode.html#configuring-schemaless-mode"/>

如上所述，有三种配置元素需要在无架构模式下使用 Solr。在 _defaultSolr 包含的配置集中，这些已经配置好了。但是，如果您想自己实现无架构，则应进行以下更改。  

### 启用托管架构<a href="http://lucene.apache.org/solr/guide/7_0/schemaless-mode.html#enable-managed-schema"/>

如 SolrConfig 中的 Schema Factory Definition 一节所述，除非您的配置指定 ClassicIndexSchemaFactory 应该使用 Managed
    Schema 支持，否则默认情况下启用 Managed Schema 支持。  
您可以通过添加显式 &lt;schemaFactory/&gt; (如下所示) 来配置 ManagedIndexSchemaFactory (并控制所使用的资源文件，或禁用将来的修改)，请参阅 SolrConfig中的架构工厂定义以获取有关可用选项的更多详细信息。  
```
&lt;schemaFactory class="ManagedIndexSchemaFactory"&gt;
  &lt;bool name="mutable"&gt;true&lt;/bool&gt;
  &lt;str name="managedSchemaResourceName"&gt;managed-schema&lt;/str&gt;
&lt;/schemaFactory&gt;
```

### 启用字段类猜测<a href="http://lucene.apache.org/solr/guide/7_0/schemaless-mode.html#enable-field-class-guessing"/>

在 Solr 中，UpdateRequestProcessorChain 定义了一系列插件，这些插件在索引之前或索引时应用于文档。  
Solr 的无架构模式的字段猜测方面使用专门定义的允许 Solr 猜测字段类型的 UpdateRequestProcessorChain。您还可以定义要使用的默认字段类型类。  
首先，您应该如下定义它（更新处理器工厂文档，请参阅下面的 javadoc 链接）：  
```
&lt;updateProcessor class="solr.UUIDUpdateProcessorFactory" name="uuid"/&gt;
  &lt;updateProcessor class="solr.RemoveBlankFieldUpdateProcessorFactory" name="remove-blank"/&gt;
  &lt;updateProcessor class="solr.FieldNameMutatingUpdateProcessorFactory" name="field-name-mutating"&gt; 【1】
    &lt;str name="pattern"&gt;[^\w-\.]&lt;/str&gt;
    &lt;str name="replacement"&gt;_&lt;/str&gt;
  &lt;/updateProcessor&gt;
  &lt;updateProcessor class="solr.ParseBooleanFieldUpdateProcessorFactory" name="parse-boolean"/&gt; 【2】
  &lt;updateProcessor class="solr.ParseLongFieldUpdateProcessorFactory" name="parse-long"/&gt;
  &lt;updateProcessor class="solr.ParseDoubleFieldUpdateProcessorFactory" name="parse-double"/&gt;
  &lt;updateProcessor class="solr.ParseDateFieldUpdateProcessorFactory" name="parse-date"&gt;
    &lt;arr name="format"&gt;
      &lt;str&gt;yyyy-MM-dd'T'HH:mm:ss.SSSZ&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd'T'HH:mm:ss,SSSZ&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd'T'HH:mm:ss.SSS&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd'T'HH:mm:ss,SSS&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd'T'HH:mm:ssZ&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd'T'HH:mm:ss&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd'T'HH:mmZ&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd'T'HH:mm&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd HH:mm:ss.SSSZ&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd HH:mm:ss,SSSZ&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd HH:mm:ss.SSS&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd HH:mm:ss,SSS&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd HH:mm:ssZ&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd HH:mm:ss&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd HH:mmZ&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd HH:mm&lt;/str&gt;
      &lt;str&gt;yyyy-MM-dd&lt;/str&gt;
    &lt;/arr&gt;
  &lt;/updateProcessor&gt;
  &lt;updateProcessor class="solr.AddSchemaFieldsUpdateProcessorFactory" name="add-schema-fields"&gt; 【3】
    &lt;lst name="typeMapping"&gt;
      &lt;str name="valueClass"&gt;java.lang.String&lt;/str&gt; 【4】
      &lt;str name="fieldType"&gt;text_general&lt;/str&gt;
      &lt;lst name="copyField"&gt; 【5】
        &lt;str name="dest"&gt;*_str&lt;/str&gt;
        &lt;int name="maxChars"&gt;256&lt;/int&gt;
      &lt;/lst&gt;
      &lt;!-- Use as default mapping instead of defaultFieldType --&gt;
      &lt;bool name="default"&gt;true&lt;/bool&gt;
    &lt;/lst&gt;
    &lt;lst name="typeMapping"&gt;
      &lt;str name="valueClass"&gt;java.lang.Boolean&lt;/str&gt;
      &lt;str name="fieldType"&gt;booleans&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="typeMapping"&gt;
      &lt;str name="valueClass"&gt;java.util.Date&lt;/str&gt;
      &lt;str name="fieldType"&gt;pdates&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="typeMapping"&gt;
      &lt;str name="valueClass"&gt;java.lang.Long&lt;/str&gt; 【6】
      &lt;str name="valueClass"&gt;java.lang.Integer&lt;/str&gt;
      &lt;str name="fieldType"&gt;plongs&lt;/str&gt;
    &lt;/lst&gt;
    &lt;lst name="typeMapping"&gt;
      &lt;str name="valueClass"&gt;java.lang.Number&lt;/str&gt;
      &lt;str name="fieldType"&gt;pdoubles&lt;/str&gt;
    &lt;/lst&gt;
  &lt;/updateProcessor&gt;
  &lt;!-- The update.autoCreateFields property can be turned to false to disable schemaless mode --&gt;
  &lt;updateRequestProcessorChain name="add-unknown-fields-to-the-schema" default="${update.autoCreateFields:true}"
           processor="uuid,remove-blank,field-name-mutating,parse-boolean,parse-long,parse-double,parse-date,add-schema-fields"&gt; 【7】
    &lt;processor class="solr.LogUpdateProcessorFactory"/&gt;
    &lt;processor class="solr.DistributedUpdateProcessorFactory"/&gt;
    &lt;processor class="solr.RunUpdateProcessorFactory"/&gt;
  &lt;/updateRequestProcessorChain&gt;
```
这条链上定义了很多东西。我们来看看其中的一些：  
此链定义将为要从相应的文本字段创建的字符串字段建立多个复制字段规则。如果您的数据导致您最终使用了大量的复制字段规则，则索引可能会明显减慢，并且索引大小将会更大。为了控制这些问题，建议您查看创建的复制字段规则，并删除不需要的 faceting、排序、突出显示等。  
  
如果您对这个链中使用的类的更多的信息感兴趣，下面是上面提到的更新处理器工厂的 Javadocs 的链接：  
  

    - [UUIDUpdateProcessorFactory](https://lucene.apache.org/solr/7_0_0//solr-core/org/apache/solr/update/processor/RemoveBlankFieldUpdateProcessorFactory.html)
    
    - [RemoveBlankFieldUpdateProcessorFactory](https://lucene.apache.org/solr/7_0_0//solr-core/org/apache/solr/update/processor/RemoveBlankFieldUpdateProcessorFactory.html)
    
    - [FieldNameMutatingUpdateProcessorFactory](https://lucene.apache.org/solr/7_0_0//solr-core/org/apache/solr/update/processor/FieldNameMutatingUpdateProcessorFactory.html)
    
    - [ParseBooleanFieldUpdateProcessorFactory](https://lucene.apache.org/solr/7_0_0//solr-core/org/apache/solr/update/processor/ParseBooleanFieldUpdateProcessorFactory.html)
    
    - [ParseLongFieldUpdateProcessorFactory](https://lucene.apache.org/solr/7_0_0//solr-core/org/apache/solr/update/processor/ParseLongFieldUpdateProcessorFactory.html)
    
    - [ParseDoubleFieldUpdateProcessorFactory](https://lucene.apache.org/solr/7_0_0//solr-core/org/apache/solr/update/processor/ParseDoubleFieldUpdateProcessorFactory.html)
    
    - [ParseDateFieldUpdateProcessorFactory](https://lucene.apache.org/solr/7_0_0//solr-core/org/apache/solr/update/processor/ParseDateFieldUpdateProcessorFactory.html)
    
    - [AddSchemaFieldsUpdateProcessorFactory](https://lucene.apache.org/solr/7_0_0//solr-core/org/apache/solr/update/processor/AddSchemaFieldsUpdateProcessorFactory.html)
    

### 设置默认的 UpdateRequestProcessorChain<a href="http://lucene.apache.org/solr/guide/7_0/schemaless-mode.html#set-the-default-updaterequestprocessorchain"/>

一旦定义了 UpdateRequestProcessorChain，你必须指示你的 UpdateRequestHandlers 在处理索引更新时使用它（即添加、删除、替换文档）。  
有两种方法可以做到这一点。上面显示的更新链有一个、default=true、属性，将用于任何更新处理程序。  
另一种更明确的方法是使用 InitParams 来设置所有/update请求处理程序的默认值：  
```
&lt;initParams path="/update/**"&gt;
  &lt;lst name="defaults"&gt;
    &lt;str name="update.chain"&gt;add-unknown-fields-to-the-schema&lt;/str&gt;
  &lt;/lst&gt;
&lt;/initParams&gt;
```
```
Tip：完成所有这些更改后，应重新启动 Solr 或重新加载内核。
```
### 禁用自动字段猜测<a href="http://lucene.apache.org/solr/guide/7_0/schemaless-mode.html#disabling-automatic-field-guessing"/>

使用 update.autoCreateFields 属性可以禁用自动字段创建。要做到这一点，您可以使用 Config API 和命令，例如：  
```
curl http://host:8983/solr/mycollection/config -d '{"set-user-property": {"update.autoCreateFields":"false"}}'
```

## 索引文件的例子<a href="http://lucene.apache.org/solr/guide/7_0/schemaless-mode.html#examples-of-indexed-documents"/>

一旦启用了无架构模式（无论是手动配置模式还是使用 _defaultconfigset），包含未在架构中定义的字段的文档将使用自动添加到架构的猜测字段类型进行索引。  
例如，添加一个 CSV 文档会导致未知的字段被添加，字段类型基于值：  
```
curl "http://localhost:8983/solr/gettingstarted/update?commit=true" -H "Content-type:application/csv" -d '
id,Artist,Album,Released,Rating,FromDistributor,Sold
44C,Old Shews,Mead for Walking,1988-08-13,0.01,14,0'
```
输出指示成功：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;&lt;int name="status"&gt;0&lt;/int&gt;&lt;int name="QTime"&gt;106&lt;/int&gt;&lt;/lst&gt;
&lt;/response&gt;
```
架构中的字段（输出自：curl http://localhost:8983/solr/gettingstarted/schema/fields）：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":2},
  "fields":[{
      "name":"Album",
      "type":"text_general"},
    {
      "name":"Artist",
      "type":"text_general"},
    {
      "name":"FromDistributor",
      "type":"plongs"},
    {
      "name":"Rating",
      "type":"pdoubles"},
    {
      "name":"Released",
      "type":"pdates"},
    {
      "name":"Sold",
      "type":"plongs"},
    {
      "name":"_root_", ...},
    {
      "name":"_text_", ...},
    {
      "name":"_version_", ...},
    {
      "name":"id", ...}
]}
```
另外字符串版本的文本字段被索引，使用 copyFields 到 *_str 动态字段:(输出：curl http://localhost:8983/solr/gettingstarted/schema/copyfields）：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":0},
  "copyFields":[{
      "source":"Artist",
      "dest":"Artist_str",
      "maxChars":256},
    {
      "source":"Album",
      "dest":"Album_str",
      "maxChars":256}]}
```
您仍然可以显式：即使您希望对大多数字段使用无架构模式，您仍然可以使用架构 API 在索引使用它们的文档之前先用创建的类型先行创建一些字段。  
在内部，架构 API 和架构更新处理器都使用相同的托管架构功能。  
此外，如果不需要文本字段的 * _str 版本，则可以从自动生成的架构中删除 copyField 定义，并且由于现在定义了原始字段，因此不会重新添加。  
一旦一个字段被添加到架构，其字段类型是固定的。因此，添加具有与先前猜测的字段类型冲突的字段值的文档将失败。例如，在添加上面的文档之后，“ Sold” 字段具有 fieldType plongs，但是下面的文档在该字段中具有非整数十进制值：  
```
curl "http://localhost:8983/solr/gettingstarted/update?commit=true" -H "Content-type:application/csv" -d '
id,Description,Sold
19F,Cassettes by the pound,4.93'
```
此文档将失败，如以下输出所示：  
```
&lt;response&gt;
  &lt;lst name="responseHeader"&gt;
    &lt;int name="status"&gt;400&lt;/int&gt;
    &lt;int name="QTime"&gt;7&lt;/int&gt;
  &lt;/lst&gt;
  &lt;lst name="error"&gt;
    &lt;str name="msg"&gt;ERROR: [doc=19F] Error adding field 'Sold'='4.93' msg=For input string: "4.93"&lt;/str&gt;
    &lt;int name="code"&gt;400&lt;/int&gt;
  &lt;/lst&gt;
&lt;/response&gt;
```
