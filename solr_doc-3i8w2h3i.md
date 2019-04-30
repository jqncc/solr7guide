## Solr查询如何实现结果分组 
Solr 结果分组将具有公共字段值的文档组分组，并返回每个组的顶部文档。   
  
例如：如果您在电子零售商的电子商务网站上搜索 “DVD”，则可能会返回 “电视和视频”，“电影”和“计算机”三个类别，每个类别有三个结果。在这种情况下，查询词 “DVD” 出现在所有三个类别中，所以 Solr 将它们组合在一起以增加用户的相关性。  
应该优先使用 Collapse 和 Expand  
Solr 的 [Collapse 和 Expand 功能](https://www.w3cschool.cn/solr_doc/solr_doc-ds692h31.html)较新，主要与结果分组重叠。这两种特性都是独一无二的，并且它们具有不同的性能特性。也就是说，在大多数情况下，“Collapse”和“Expand”优于“结果分组（Result Grouping）”。  
结果分组与分面（[Faceting](https://www.w3cschool.cn/solr_doc/solr_doc-zisb2gvv.html)）是不同的。尽管它们可能在概念上是相似的，但分面将返回所有相关的结果，并允许用户根据分面类别对结果进行细化。例如，如果您在鞋类零售商的电子商务网站上搜索“鞋子”，则 Solr 将返回该查询字词的所有结果以及诸如“尺寸”、“颜色”、“品牌”等的可选方面。  
  
但是，您可以将分组与分面相结合。分组的 faceting 支持 facet.field，facet.range，但目前不支持日期和支点 faceting。facet 计数是根据第一个 group.field 参数计算的，而其他group.field 参数则被忽略。  
已分组的分面与没有分组的分面不同，(sum of all facets) == (total of products with that property)，如以下示例所示：  
对象1  

    - name：Phaser 4620a
    - ppm：62
    - product_range：6

对象2  

    - name：Phaser 4620i
    - ppm：65
    - product_range：6

对象3  

    - name：ML6512
    - ppm：62
    - product_range：7
如果您要求 Solr 将这些文档按 “product_range” 进行分组，则组的总数量为2，但 ppm 的分面数是：62为2，65为1。  

## 分组参数<a href="http://lucene.apache.org/solr/guide/7_0/result-grouping.html#grouping-parameters"/>

结果分组采用以下请求参数。任何数量的这些请求参数都可以包含在一个请求中：  

**group 参数**
    
        如果为<code>true</code>，查询结果将被分组。  
    
**group.field 参数**
    
        结果分组字段的名称。该字段必须是单值的，并且要么是索引的，要么是具有值源的字段类型，并且在函数查询中工作，例如<code>ExternalFileField</code>。它也必须是基于字符串的字段，如<code>StrField</code>或者<code>TextField</code>
          
    
**group.func 参数**
    
        根据函数查询的唯一值进行分组。  
    
此选项不适用于分布式搜索。  
  
**group.query 参数**
    
        返回与给定查询匹配的一组文档。  
    
**rows 参数**
    
        要返回的组的数量。默认值是<code>10</code>。  
    
**start 参数**
    
        指定组列表的初始偏移量。  
    
**group.limit 参数**
    
        指定每个组返回的结果数量。默认值是<code>1</code>。  
    
**group.offset 参数**
    
        指定每个组的文档列表的初始偏移量。  
    
**sort 参数**
    指定 Solr 如何对各组进行相对排序。例如：<code>sort=popularity desc</code>将导致组按照每个组中最高的受欢迎度文档进行排序。默认值是<code>score desc</code>。  
  
**group.sort 参数**
    
        指定 Solr 如何对每个组中的文档进行排序。如果<code>group.sort</code>未指定，则默认行为是使用与<code>sort</code>参数相同的有效值。  
    
**group.format 参数**
    如果将此参数设置为<code>simple</code>，则分组的文档将显示在单个平面列表中，<code>start</code>和<code>rows</code>参数会影响文档数量而不是组数量。这个参数的另一个值是<code>grouped</code>。  
  
**group.main 参数**
    
        如果为<code>true</code>，则将第一个字段分组命令的结果被用作响应中的主要结果列表，使用<code>group.format=simple</code>。  
**group.ngroups 参数**
    
        如果为<code>true</code>，则 Solr 包含结果中与查询匹配的组数。默认值是 false。  
        
            使用分片索引时，请参阅下面的“分布式结果分组注意事项”。  
  
**group.truncate 参数**
    
        如果为<code>true</code>，则 facet 计数基于与查询匹配的每个组的最相关文档。默认值是<code>false</code>。  
**group.facet 参数**
    确定是否为在 facet.field 参数中指定的字段分面计算分组的分面。根据第一个指定的组计算分组的分面。和正常的字段分面一样，字段不应该被标记（否则计算每个标记的计数）。分组的分面支持单个和多值字段。默认是<code>false</code>。  
  

        
            这个选项可能会有很高的性能成本。  
  
        
        
            使用分片索引时，请参阅下面的“分布式结果分组注意事项”。  
  
  
**group.cache.percent 参数**
    如果将此参数设置为大于0的数字，则可以对结果分组进行缓存。结果分组执行两次搜索；这个选项缓存第二个搜索。默认值是<code>0</code>，最大值是<code>100</code>。  
  

        
            测试表明，组缓存仅通过布尔、通配符和模糊查询来提高搜索时间。对于像术语或“全部匹配”查询这样的简单查询，组缓存会降低性能。  
          
    

任何数量的组命令（例如，group.field，group.func，group.query，等等）可以在单个请求中指定。  

## 分组示例<a href="http://lucene.apache.org/solr/guide/7_0/result-grouping.html#grouping-examples"/>

以下所有示例查询均适用于 Solr 的 “bin / solr -e techproducts” 示例。  

### 按字段进行结果分组<a href="http://lucene.apache.org/solr/guide/7_0/result-grouping.html#grouping-results-by-field"/>

在此示例中，我们将根据 manu_exact 字段对结果进行分组，该字段指定样本数据集中项目的制造商。  
```
http://localhost:8983/solr/techproducts/select?fl=id,name&amp;q=solr+memory&amp;group=true&amp;group.field=manu_exact
```
```
{
"..."
"grouped":{
  "manu_exact":{
    "matches":6,
    "groups":[{
        "groupValue":"Apache Software Foundation",
        "doclist":{"numFound":1,"start":0,"docs":[
            {
              "id":"SOLR1000",
              "name":"Solr, the Enterprise Search Server"}]
        }},
      {
        "groupValue":"Corsair Microsystems Inc.",
        "doclist":{"numFound":2,"start":0,"docs":[
            {
              "id":"VS1GB400C3",
              "name":"CORSAIR ValueSelect 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - Retail"}]
        }},
      {
        "groupValue":"A-DATA Technology Inc.",
        "doclist":{"numFound":1,"start":0,"docs":[
            {
              "id":"VDBDB1A16",
              "name":"A-DATA V-Series 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - OEM"}]
        }},
      {
        "groupValue":"Canon Inc.",
        "doclist":{"numFound":1,"start":0,"docs":[
            {
              "id":"0579B002",
              "name":"Canon PIXMA MP500 All-In-One Photo Printer"}]
        }},
      {
        "groupValue":"ASUS Computer Inc.",
        "doclist":{"numFound":1,"start":0,"docs":[
            {
              "id":"EN7800GTX/2DHTV/256M",
              "name":"ASUS Extreme N7800GTX/2DHTV (256 MB)"}]
        }
      }]}}}
```
响应表明我们的查询总共有六个匹配项。对于 group.field 的五个唯一值中的每一个，Solr 返回一个 groupValue 的 docList，使得 numFound 指示在该组中的文件的总数，并根据隐式默认 group.limit=1 和 group.sort=score desc 参数返回顶级文档。然后根据隐式 sort=score desc，按照每个组内顶部文档的得分对结果组进行排序，返回的组数限于隐式 rows=10。  
  
我们可以使用请求参数 group.main=true 运行相同的查询。这将把结果格式化为一个单一的文档列表。这种平面格式不包含像正常结果分组查询结果那样多的信息 - 特别是每个组中的 numFound - 但是现有的 Solr 客户端可能更容易解析。  
```
http://localhost:8983/solr/techproducts/select?fl=id,name,manufacturer&amp;q=solr+memory&amp;group=true&amp;group.field=manu_exact&amp;group.main=true
```
```
{
  "responseHeader":{
    "status":0,
    "QTime":1,
    "params":{
      "fl":"id,name,manufacturer",
      "indent":"true",
      "q":"solr memory",
      "group.field":"manu_exact",
      "group.main":"true",
      "group":"true"}},
  "grouped":{},
  "response":{"numFound":6,"start":0,"docs":[
      {
        "id":"SOLR1000",
        "name":"Solr, the Enterprise Search Server"},
      {
        "id":"VS1GB400C3",
        "name":"CORSAIR ValueSelect 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - Retail"},
      {
        "id":"VDBDB1A16",
        "name":"A-DATA V-Series 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - OEM"},
      {
        "id":"0579B002",
        "name":"Canon PIXMA MP500 All-In-One Photo Printer"},
      {
        "id":"EN7800GTX/2DHTV/256M",
        "name":"ASUS Extreme N7800GTX/2DHTV (256 MB)"}]
  }
}
```

### 按查询分组<a href="http://lucene.apache.org/solr/guide/7_0/result-grouping.html#grouping-by-query"/>

在这个例子中，我们将使用这个 group.query 参数在两个不同的价格范围内找到 “memory” 的前三个结果：0.00 到 99.99，以及超过100。  
```
http://localhost:8983/solr/techproducts/select?indent=true&amp;fl=name,price&amp;q=memory&amp;group=true&amp;group.query=price:0+TO+99.99&amp;group.query=price:[100+TO+*]&amp;group.limit=3
```
```
{
  "responseHeader":{
    "status":0,
    "QTime":42,
    "params":{
      "fl":"name,price",
      "indent":"true",
      "q":"memory",
      "group.limit":"3",
      "group.query":["price:[0 TO 99.99]",
      "price:[100 TO *]"],
      "group":"true"}},
  "grouped":{
    "price:[0 TO 99.99]":{
      "matches":5,
      "doclist":{"numFound":1,"start":0,"docs":[
          {
            "name":"CORSAIR ValueSelect 1GB 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) System Memory - Retail",
            "price":74.99}]
      }},
    "price:[100 TO *]":{
      "matches":5,
      "doclist":{"numFound":3,"start":0,"docs":[
          {
            "name":"CORSAIR  XMS 2GB (2 x 1GB) 184-Pin DDR SDRAM Unbuffered DDR 400 (PC 3200) Dual Channel Kit System Memory - Retail",
            "price":185.0},
          {
            "name":"Canon PIXMA MP500 All-In-One Photo Printer",
            "price":179.99},
          {
            "name":"ASUS Extreme N7800GTX/2DHTV (256 MB)",
            "price":479.95}]
      }
    }
  }
}
```
在这种情况下，Solr 找到了五个“memory”匹配，但只返回四个按价格分组的结果。这是因为“memory”的一个结果没有分配给它的价格。  

## 分布式结果分组警告<a href="http://lucene.apache.org/solr/guide/7_0/result-grouping.html#distributed-result-grouping-caveats"/>

分布式搜索支持分组，但是有一些注意事项：  
  

    - 目前 group.func 在任何分布式搜索中都不支持
    - group.ngroups 和 group.facet 要求每个组中的所有文档必须共同位于同一分片上，以便返回准确的计数。在许多情况下，通过组合键进行文件路由可能是有用的解决方案
