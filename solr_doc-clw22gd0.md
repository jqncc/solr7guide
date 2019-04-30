## Solr转换和索引自定义JSON 
<div class="content-intro view-box ">如果您想要索引 JSON 文档而不将其转换为 Solr 的结构，则可以通过包含一些带有更新请求的参数将它们添加到 Solr。这些参数提供了如何将单个 JSON 文件拆分为多个 Solr 文档以及如何将字段映射到 Solr 的架构的信息。一个或多个有效的 JSON 文档可以通过配置参数发送到 /update/json/docs 路径。
      
  

## 映射参数

这些参数允许您定义如何为多个 Solr 文档读取 JSON 文件。
      
  

<dt>split  
</dt>
    
        定义将输入 JSON 拆分为多个 Solr 文档的路径，如果您在单个 JSON 文件中有多个文档，则需要该路径。如果整个 JSON 生成一个 solr 文档，路径必须是“<code>/</code>”。可以通过用管道<code>(|)</code>分隔它们来传递多个分割路径，示例：<code>split=/|/foo|/foo/bar</code>。如果一个路径是另一个路径的子路径，则它们自动成为子文档  
    
<dt>f  
</dt>
    
        多值映射参数。参数的格式是<code>target-field-name:json-path<font color="#000000" face="Verdana, Arial, Helvetica, sans-serif"><span style="white-space: normal; background-color: rgb(255, 255, 255);">；</span></font></code><code>json-path</code>是必需的；<code>target-field-name</code>是
            Solr 文档字段名称，并且是可选的。如果未指定，则从输入 JSON 自动派生。默认的目标字段名称是字段的完全限定名称。  
        
            这里可以使用通配符，请参阅下面的使用通配符字段名称获取更多信息。  
          
    
**mapUniqueKeyOnly**

    
        （boolean 值）当输入 JSON 中的字段在模式中不可用且无架构模式未启用时，此参数特别方便。这会将所有字段编入默认搜索字段（使用下面的 df 参数），只将<code>uniqueKey</code>字段映射到模式中相应的字段。如果输入的 JSON 没有该<code>uniqueKey</code>字段的值，则会为此生成一个 UUID。  
    
<dt>df  
</dt>
    
        如果使用该<code>mapUniqueKeyOnly</code>标志，则更新处理程序需要一个数据应该被索引到的字段。这是其他处理程序用作默认搜索字段的相同字段。  
    
**srcField**

    
        这是 JSON 源将存储到的字段的名称。这只能用于<code>split=/</code>（即，您希望您的 JSON 输入文件索引为一个单一的 Solr 文档）。请注意，原子更新将导致该字段与文档不同步。  
    
<dt>echo  
</dt>
    
        这仅用于调试目的。将其设置为<code>true</code>如果您希望将文档作为响应返回。什么都不会被索引。  
    

例如，如果我们有一个包含两个文档的 JSON 文件，我们可以像这样定义一个更新请求：  
```
curl 'http://localhost:8983/solr/my_collection/update/json/docs'\
'?split=/exams'\
'&amp;f=first:/first'\
'&amp;f=last:/last'\
'&amp;f=grade:/grade'\
'&amp;f=subject:/exams/subject'\
'&amp;f=test:/exams/test'\
'&amp;f=marks:/exams/marks'\
 -H 'Content-type:application/json' -d '
{
  "first": "John",
  "last": "Doe",
  "grade": 8,
  "exams": [
    {
      "subject": "Maths",
      "test"   : "term1",
      "marks"  : 90},
    {
      "subject": "Biology",
      "test"   : "term1",
      "marks"  : 86}
  ]
}'
```

您可以通过使用请求参数来存储和重用参数。  
```
 curl http://localhost:8983/solr/my_collection/config/params -H 'Content-type:application/json' -d '{
 "set": {
 "my_params": {
 "split": "/exams",
 "f": ["first:/first","last:/last","grade:/grade","subject:/exams/subject","test:/exams/test"]
 }}}'
```

并使用它，如下所示：  
```
curl 'http://localhost:8983/solr/my_collection/update/json/docs?useParams=my_params' -H 'Content-type:application/json' -d '{
"first": "John",
"last": "Doe",
"grade": 8,
"exams": [
{
"subject": "Maths",
"test" : "term1",
"marks" : 90},
{
"subject": "Biology",
"test" : "term1",
"marks" : 86}
]
}'
```

有了这个要求，我们已经定义“exams”包含多个文件。另外，我们已经将输入文档中的几个字段映射到 Solr 字段。  
更新请求完成后，以下两个文件将被添加到索引中：  
```
{
  "first":"John",
  "last":"Doe",
  "marks":90,
  "test":"term1",
  "subject":"Maths",
  "grade":8
}
{
  "first":"John",
  "last":"Doe",
  "marks":86,
  "test":"term1",
  "subject":"Biology",
  "grade":8
}
```

在之前的例子中，我们想要在 Solr 中使用的所有字段与在输入 JSON 中使用的名称相同。如果是这种情况，我们可以简化请求，如下所示：  
```
curl 'http://localhost:8983/solr/my_collection/update/json/docs'\
'?split=/exams'\
'&amp;f=/first'\
'&amp;f=/last'\
'&amp;f=/grade'\
'&amp;f=/exams/subject'\
'&amp;f=/exams/test'\
'&amp;f=/exams/marks'\
 -H 'Content-type:application/json' -d '
{
  "first": "John",
  "last": "Doe",
  "grade": 8,
  "exams": [
    {
      "subject": "Maths",
      "test"   : "term1",
      "marks"  : 90},
    {
      "subject": "Biology",
      "test"   : "term1",
      "marks"  : 86}
  ]
}'
```

在这个例子中，我们简单地命名字段路径（如 /exams/test）。Solr 将自动尝试将字段的内容从 JSON 输入添加到具有相同名称的字段中的索引。  
Note：如果索引之前的架构中不存在文档，文档将被拒绝。所以，如果您不使用无架构模式，请预先创建这些字段。如果您在无架构模式下工作，则不存在的字段将以 Solr 对字段类型的最佳猜测而创建。
      
  

## 对字段名使用通配符

可以指定通配符来自动映射字段, 而不是显式指定所有字段名称。
      
  
但是这有两个限制：通配符只能在 json 路径的末尾使用；分割路径不能使用通配符。  
一个星号 * 只映射到直接的子级，双星号 \*\* 递归地映射到所有的后代。以下是通配符路径映射示例：  

    - f=$FQN:/**：将所有字段映射到 JSON 字段的完全限定名 ($FQN)。完全限定名是通过连接层次结构中的所有关键字（以句点（.））作为分隔符获得的。如果未指定 f 路径映射，则这是默认行为。
    - f=/docs/*：将文档中的所有字段以 json 中给出的名称映射。
    - f=/docs/**：将文档中的所有字段及其子节点映射为 json 中给出的名称。
    - f=searchField:/docs/* ：将 /docs 下的所有字段映射到一个名为 “searchField” 的字段。
    - f=searchField:/docs/** ：将 /docs 及其子项下的所有字段映射到 searchField。

使用通配符我们可以进一步简化前面的例子，如下所示：  
```
curl 'http://localhost:8983/solr/my_collection/update/json/docs'\
'?split=/exams'\
'&amp;f=/**'\
 -H 'Content-type:application/json' -d '
{
  "first": "John",
  "last": "Doe",
  "grade": 8,
  "exams": [
    {
      "subject": "Maths",
      "test"   : "term1",
      "marks"  : 90},
    {
      "subject": "Biology",
      "test"   : "term1",
      "marks"  : 86}
  ]
}'
```

因为我们希望在 JSON 输入中找到字段时使用字段名进行索引，所以双重通配符 f=/** 将把所有字段及其后代映射到 Solr 中的相同字段。
      
  
也可以将所有值发送到单个字段，然后对其进行全文搜索。这是一个很好的选择，可以盲目索引和查询 JSON 文档，而不必担心字段和架构。  
```
curl 'http://localhost:8983/solr/my_collection/update/json/docs'\
'?split=/'\
'&amp;f=txt:/**'\
 -H 'Content-type:application/json' -d '
{
  "first": "John",
  "last": "Doe",
  "grade": 8,
  "exams": [
    {
      "subject": "Maths",
      "test"   : "term1",
      "marks"  : 90},
    {
      "subject": "Biology",
      "test"   : "term1",
      "marks"  : 86}
  ]
}'
```

在上面的例子中，我们已经说过所有的字段都应该添加到名为 “txt” 的 Solr 中的一个字段中。这会将多个字段添加到单个字段，因此您选择的任何字段都应该是多值的。
      
  
默认行为是使用节点的完全限定名称（FQN）。所以，如果我们不定义任何字段映射，像这样：  
```
curl 'http://localhost:8983/solr/my_collection/update/json/docs?split=/exams'\
    -H 'Content-type:application/json' -d '
{
  "first": "John",
  "last": "Doe",
  "grade": 8,
  "exams": [
    {
      "subject": "Maths",
      "test"   : "term1",
      "marks"  : 90},
    {
      "subject": "Biology",
      "test"   : "term1",
      "marks"  : 86}
  ]
}'
```

索引文档将被添加到索引中，其字段如下所示：  
```
{
  "first":"John",
  "last":"Doe",
  "grade":8,
  "exams.subject":"Maths",
  "exams.test":"term1",
  "exams.marks":90},
{
  "first":"John",
  "last":"Doe",
  "grade":8,
  "exams.subject":"Biology",
  "exams.test":"term1",
  "exams.marks":86}
```


## 单个有效载荷中的多个文档

此功能支持 JSON 行格式（.jsonl）中的文档，每行指定一个文档。  
例如：  
```
curl 'http://localhost:8983/solr/my_collection/update/json/docs' -H 'Content-type:application/json' -d '
{ "first":"Steve", "last":"Jobs", "grade":1, "subject": "Social Science", "test" : "term1", "marks" : 90}
{ "first":"Steve", "last":"Woz", "grade":1, "subject": "Political Science", "test" : "term1", "marks" : 86}'
```

甚至还有一些文档，如下例所示：  
```
curl 'http://localhost:8983/solr/my_collection/update/json/docs' -H 'Content-type:application/json' -d '[
{ "first":"Steve", "last":"Jobs", "grade":1, "subject": "Computer Science", "test"   : "term1", "marks"  : 90},
{ "first":"Steve", "last":"Woz", "grade":1, "subject": "Calculus", "test"   : "term1", "marks"  : 86}]'
```


## 索引嵌套文档

以下是索引嵌套文档的示例：  
```
curl 'http://localhost:8983/solr/my_collection/update/json/docs?split=/|/orgs'\
    -H 'Content-type:application/json' -d '{
  "name": "Joe Smith",
  "phone": 876876687,
  "orgs": [
    {
      "name": "Microsoft",
      "city": "Seattle",
      "zip": 98052
    },
    {
      "name": "Apple",
      "city": "Cupertino",
      "zip": 95014
    }
  ]
}'
```
在这个例子中，索引的文档如下所示：  
```
{
  "name":"Joe Smith",
  "phone":876876687,
  "_childDocuments_":[
    {
      "name":"Microsoft",
      "city":"Seattle",
      "zip":98052},
    {
      "name":"Apple",
      "city":"Cupertino",
      "zip":95014}]}
```


## 自定义 JSON 索引的技巧

1 <li>无架构模式：自动处理字段创建。字段猜测可能不完全按照您的预期，但它的工作原理。最好的办法是以无架构模式设置本地服务器，索引一些示例文档，并在索引之前使用适当的字段类型在实际设置中创建这些字段。</li>2 <li>预先创建的架构：将文档发布到 /update/json/docs 端点，并且 echo=true。这将为您提供需要创建的字段名称的列表。在实际编制索引之前创建这些字段</li>3 <li>没有架构，只有全文搜索：所有你需要做的就是在您的 JSON 上进行全文搜索。按照 “设置 JSON 默认值”部分中的设置配置。</li>可以将任何 json 发送到 /update/json/docs 端点，组件的默认配置如下：  
```
&lt;initParams path="/update/json/docs"&gt;
  &lt;lst name="defaults"&gt;
    &lt;!-- this ensures that the entire json doc will be stored verbatim into one field --&gt;
    &lt;str name="srcField"&gt;_src_&lt;/str&gt;
    &lt;!-- This means a the uniqueKeyField will be extracted from the fields and
         all fields go into the 'df' field. In this config df is already configured to be 'text'
     --&gt;
    &lt;str name="mapUniqueKeyOnly"&gt;true&lt;/str&gt;
    &lt;!-- The default search field where all the values are indexed to --&gt;
    &lt;str name="df"&gt;text&lt;/str&gt;
  &lt;/lst&gt;
&lt;/initParams&gt;
```

所以，如果没有参数传递，整个 json 文件将被索引到该 _src_ 字段，并且输入 JSON 中的所有值都将转到名为 text 的字段。如果存在 uniqueKey 的值，则存储该值，如果不能从输入的 JSON 获取值，则创建 UUID 并将其用作 uniqueKey 字段值。  
  
或者，使用“请求参数”功能设置这些参数：  
```
 curl http://localhost:8983/solr/my_collection/config/params -H 'Content-type:application/json' -d '{
 "set": {
 "full_txt": {
     "srcField": "_src_",
     "mapUniqueKeyOnly" : true,
     "df": "text"
 }}}'
```

使用每个请求发送参数 useParams = full_txt。  
