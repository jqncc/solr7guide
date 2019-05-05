## Solr使用枚举字段 
<div class="content-intro view-box ">EnumFieldType 允许定义一个字段，其值是一个封闭的集合，排序顺序是预先确定的，但不是字母或数字。这是严重性列表或风险定义的示例。  
  
注意：EnumField 已被弃用；EnumField 已被弃用，以支持 EnumFieldType；下面的所有配置示例都使用EnumFieldType。  

## 在schema.xml中定义一个EnumFieldType<a href="http://lucene.apache.org/solr/guide/7_0/working-with-enum-fields.html#defining-an-enumfieldtype-in-schema-xml"/>

EnumFieldType 类型的定义非常简单，就像在本例中定义 “priorityLevel” 和 “riskLevel” 枚举字段类型一样：  
```
&lt;fieldType name="priorityLevel" class="solr.EnumFieldType" docValues="true" enumsConfig="enumsConfig.xml" enumName="priority"/&gt;
&lt;fieldType name="riskLevel"     class="solr.EnumFieldType" docValues="true" enumsConfig="enumsConfig.xml" enumName="risk"    /&gt;
```
除了所有字段类型所共有的 name 和 class，这个类型还需要两个额外的参数：  

**enumsConfig**
    
        包含 &lt;enum/&gt; 字段值列表的配置文件的名称，以及您希望与此字段类型一起使用的顺序。如果未定义指定文件的路径，则该文件应位于 conf 集合的目录中。  
    
**enumName**
    
        enumsConfig 用于此类型的文件中特定枚举的名称。  
    

请注意，docValues="true" 必须在 EnumFieldType 字段类型或字段说明中指定。  

## 定义 EnumFieldType 配置文件<a href="http://lucene.apache.org/solr/guide/7_0/working-with-enum-fields.html#defining-the-enumfieldtype-configuration-file"/>

如果在您的 Solr 架构中有多个用于枚举的使用，则使用 enumsConfig 参数命名的文件可以包含多个具有不同名称的枚举值列表。  
在此示例中，定义了两个值列表。每个列表在枚举开始和结束标签之间:  
```
&lt;?xml version="1.0" ?&gt;
&lt;enumsConfig&gt;
  &lt;enum name="priority"&gt;
    &lt;value&gt;Not Available&lt;/value&gt;
    &lt;value&gt;Low&lt;/value&gt;
    &lt;value&gt;Medium&lt;/value&gt;
    &lt;value&gt;High&lt;/value&gt;
    &lt;value&gt;Urgent&lt;/value&gt;
  &lt;/enum&gt;
  &lt;enum name="risk"&gt;
    &lt;value&gt;Unknown&lt;/value&gt;
    &lt;value&gt;Very Low&lt;/value&gt;
    &lt;value&gt;Low&lt;/value&gt;
    &lt;value&gt;Medium&lt;/value&gt;
    &lt;value&gt;High&lt;/value&gt;
    &lt;value&gt;Critical&lt;/value&gt;
  &lt;/enum&gt;
&lt;/enumsConfig&gt;
```
  
```
Tip：更改值：您无法在 &lt;enum/&gt; 没有重新索引的情况下更改顺序或删除现有值。但是，您可以将新值添加到末尾。
```
