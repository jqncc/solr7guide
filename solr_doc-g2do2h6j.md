## Solr空间搜索 
<div class="content-intro view-box ">Solr 支持用于空间/地理空间搜索的位置数据。  
  
使用空间搜索，您可以：  

    - 索引点或其他形状
    - 按边框或圆形或其他形状过滤搜索结果
    - 根据点之间的距离或矩形之间的相对面积来排序或提高评分
    - 生成 heatmap 生成或点绘图的 facet 计数的二维网格。

有以下四种可用于空间搜索的主要字段类型：  

    - LatLonPointSpatialField
    - LatLonType （现已弃用）以及它的 non-geodetic twin PointType
    - SpatialRecursivePrefixTreeFieldType（简称 RPT），包括衍生的 RptWithGeometrySpatialField
    - BBoxField

LatLonPointSpatialField 是 lat-lon（经纬度）点数据最常见的使用示例的理想字段类型。它取代了为了向后兼容而仍然存在的 LatLonType。对于更高级/自定义用例和多边形和 heatmaps 等选项，RPT 提供了更多的功能。  
  
RptWithGeometrySpatialField 是用于索引和搜索非点数据，尽管它也可以做点。它不能做 sort/boost。  
BBoxField 用于索引边界框、通过框进行查询、指定搜索谓词（相交、内部、包含、不相交、等于）以及相关性 sort/boost，类似 overlapRatio 或简单的区域。  

## LatLonPointSpatialField<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#latlonpointspatialfield"/>

以下是 LatLonPointSpatialField（LLPSF）通常在架构中配置的方式：  
```
&lt;fieldType name="location" class="solr.LatLonPointSpatialField" docValues="true"/&gt;
```
LLPSF 支持来回切换 indexed、stored、docValues 和 multiValued。LLPSF 在启用 “indexed”（默认）时内部使用二维 Lucene “Points”（BDK 树）索引。当启用 “docValues” 时，经纬度对被交织成 64 位，并放入 Lucene DocValues 中。docValues 数据的精确度约为一厘米。  
  

## 索引点<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#indexing-points"/>

为了索引大地测量点（纬度和经度），通过“lat，lon”顺序（逗号分隔）提供。  
  
对于索引 non-geodetic 点，这取决于：如果为 RPT 使用 x y（中间有空格）；但是，对于 PointType，请使用 x，y（使用逗号分隔）。  
如果您想使用标准的行业格式，Solr 支持 WKT 和 GeoJSON。然而它比这些简单数据的原始坐标大得多。（不赞成使用 LatLonType 或 PointType）  

## 使用查询解析器进行搜索<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#searching-with-query-parsers"/>

有两个用于地理空间搜索的空间 Solr “查询解析器”：geofilt和bbox。他们采取以下参数：  

**d 参数**
    
        径向距离，通常以公里为单位。RPT 和 BBoxField 可以通过设置<code>distanceUnits</code>来设置其他单位。  
    
**pt 参数**
    
        如果纬度和经度为中心，则使用格式“lat，lon”。否则，使用 PointType 的 “x，y” 或 RPT 字段类型的“x y”。  
    
**sfield 参数**
    
        空间索引字段。  
    
**score 参数**
    
        （高级选项；LatLonType（不建议使用）或 PointType 不支持）如果查询用于记分环境（例如作为主查询<code>q</code>），则此本地参数确定将生成什么分数。有效值是：  
        
            
                - 
                    <code>none</code>：固定分数为1.0。（默认）  
                
                - 
                    <code>kilometers</code>：字段值与指定中心点之间的距离（以公里为单位）  

                - 
                    <code>miles</code>：字段值与指定中心点之间的距离（以英里为单位）  
                
                - 
                    <code>degrees</code>：字段值与指定中心点之间的距离（以度为单位）  
                
                - 
                    <code>distance</code>：字段值与为此字段配置的<code>distanceUnits</code>中的指定中心点之间的距离  
                
                - 
                    <code>recipDistance</code>：1 / 距离  
```
注意：不要将其用于索引的非点形状 (例如多边形)。结果将是错误的。而对于RPT，只建议多值点数据，只推荐使用它，因为实现不能很好地进行扩展, 而且对于单参数字段，您应该改用单独的非 RPT 字段来进行距离排序。
```
                      
                    
                        使用<code>BBoxField</code>时，支持下列的选项：  
                      
                
                - 
                    <code>overlapRatio</code>：索引形状和查询形状之间的相对重叠。  
                
                - 
                    <code>area</code>： haversine 基于为该字段配置的<code>distanceUnits</code>表示的重叠形状的区域  
                
                - 
                    <code>area2D</code>：基于为该字段配置的<code>distanceUnits</code>表示的重叠形状的笛卡尔坐标区域  
                
            
          
    
**filter 参数**
    
        （高级选项；LatLonType（不建议使用）或 PointType 不支持）。如果您只希望查询得分（使用上述<code>score</code>本地参数），而不是过滤，则将此本地参数设置为 false。  
    


### geofilt 过滤器<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#geofilt"/>

geofilt 过滤器可以根据地理空间距离（又名“大圈距离”）从给定的点来检索结果。另一种看待它的方式是创建一个圆形的形状滤镜。例如，要查找给定经纬度五公里内的所有文件，可以输入：&amp;q=:&amp;fq={!geofilt sfield=store}&amp;pt=45.15,-93.85&amp;d=5。此过滤器在初始点周围的给定半径的圆内返回所有结果：  
  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171201/1512108981282059.png)
  

### bbox 过滤器<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#bbox"/>

bbox 过滤器与 geofilt 非常相似，只是它使用要计算的圆的边界框。请参阅下图中的蓝色框。它采用与 geofilt 相同的参数。  
  
以下是一个示例查询：  
```
&amp;q=:&amp;fq={!bbox sfield=store}&amp;pt=45.15,-93.85&amp;d=5
```
矩形形状的计算速度更快，所以它有时被用作替代 geofilt 当它可以接受的返回点以外的半径。但是，如果理想的目标是一个圆圈，但是您希望它运行得更快，那么请考虑使用 RPT 字段并尝试一个大的 distErrPct 值，比如0.1（10％半径）。这将返回半径之外的结果，但它将在形状周围稍微均匀地进行。  
<p style="text-align: center; ">![solr](https://7n.w3cschool.cn/attachments/image/20171201/1512109202330301.png)
  
当边界框包含一个极点时，边界框最终成为一个 "边界碗" (一个球形的帽子)，它包含了在圆的最低纬度以北的所有值，如果它触及北极 (或者如果它触及南极的最高纬度的南部)。  

### 通过任意矩形过滤<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#filtering-by-an-arbitrary-rectangle"/>

有时，空间搜索要求需要在矩形区域中查找所有内容，例如用户正在查看的地图所覆盖的区域。对于这种情况，geofilt 和 bbox 不会将其切割。这是一个窍门，但是您可以使用 Solr 的范围查询语法，方法是将左下角作为范围的开始，右上角作为范围的结束。  
  
下面是一个例子：  
```
&amp;q=:&amp;fq=store:[45,-94 TO 46,-93]
```
LatLonType（不建议使用）不支持跨越国际日期变更线的矩形。对于 RPT 和 BBoxField，如果是非地理空间坐标（geo="false"），那么必须引用由于空间的点，例如"x y"。  

### 优化：是否缓存<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#optimizing-cache-or-not"/>

将空间查询放入 “fq” 参数中是最常见的过滤器查询。默认情况下，Solr 会将查询缓存在过滤器缓存中。  
  
如果您知道过滤器查询（无论是否是空间）是相当唯一的，并且不太可能获得缓存命中，则指定 cache="false" 为本地参数，如以下示例所示。LatLonPointSpatialField 和 LatLonType（不建议使用）是唯一受益于此技术的空间类型。在该字段上启用 docValues（如果它尚未启用）。LatLonType（不建议使用）还需要一个 cost="100"（或多个）本地参数。  
```
&amp;q=…​mykeywords…​&amp;fq=…​someotherfilters…​&amp;fq={!geofilt cache=false}&amp;sfield=store&amp;pt=45.15,-93.85&amp;d=5
```
LLPSF 不支持 Solr 的 “PostFilter”。  

## 距离 sort 或 boost（函数查询）<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#distance-sorting-or-boosting-function-queries"/>

有四个距离函数查询：  
  

    - geodist：见下文，通常是最合适的
    - dist：计算多维向量之间的 p-norm 距离
    - hsin：计算球体上两点之间的距离
    - sqedist：计算两点之间的平方欧几里得距离

有关这些功能的查询的详细信息，请参阅函数查询部分。  

### geodist 距离函数<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#geodist"/>

geodist 是一个距离函数，需要三个可选参数：(sfield、latitude、longitude)。您可以使用 geodist 函数按距离或分数返回结果对结果进行排序。  
  
例如，要按距离递增对结果进行排序，请输入：  
```
…​&amp;q=:&amp;fq={!geofilt}&amp;sfield=store&amp;pt=45.15,-93.85&amp;d=50&amp;sort=geodist() asc
```
要将距离作为文档分数返回，请输入：  
```
…​&amp;q={!func}geodist()&amp;sfield=store&amp;pt=45.15,-93.85&amp;sort=score+asc。
```

## Solr空间搜索示例<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#more-spatial-search-examples"/>

以下是更多关于 Solr 空间搜索的有用的例子。  

### 用作子查询来展开搜索结果<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#use-as-a-sub-query-to-expand-search-results"/>

我们将在 Jacksonville、Florida 查询结果，或在距离 45.15，-93.85（Buffalo, Minnesota 附近）50公里内查询结果：  
```
&amp;q=:&amp;fq=(state:"FL" AND city:"Jacksonville") OR {!geofilt}&amp;sfield=store&amp;pt=45.15,-93.85&amp;d=50&amp;sort=geodist()+asc
```

### 通过距离分面（facet）<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#facet-by-distance"/>

通过距离分面，您可以使用 Frange 查询解析器：  
```
&amp;q=:&amp;sfield=store&amp;pt=45.15,-93.85&amp;facet.query={!frange l=0 u=5}geodist()&amp;facet.query={!frange l=5.001 u=3000}geodist()
```
还有其他方法可以做到这一点，比如在每个 facet.query 中使用：\ {！geofilt}。  

### 提高最近的结果<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#boost-nearest-results"/>

使用 [DisMax](https://www.w3cschool.cn/solr_doc/solr_doc-vpyf2gn1.html) 或 [Extended DisMax](https://www.w3cschool.cn/solr_doc/solr_doc-usk22gqk.html)，您可以将空间搜索与提升功能结合起来，以提高最近的结果：  
```
&amp;q.alt=:&amp;fq={!geofilt}&amp;sfield=store&amp;pt=45.15,-93.85&amp;d=50&amp;bf=recip(geodist(),2,200,20)&amp;sort=score desc
```

## RPT<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#rpt"/>

RPT 是指 SpatialRecursivePrefixTreeFieldType（简称 RPT）和 扩展版本 RptWithGeometrySpatialField（又名 RPT 几何）。RPT 提供了比 LatLonPointSpatialField 更多的功能改进：  

    - 非大地测量（Non-geodetic）：geo = false，一般x和y（不是经度和纬度）
    - 除了圆形和矩形之外，还可以通过多边形和其他复杂形状进行查询
    - 能够索引非点形状（如多边形）以及点 - 请参阅 RptWithGeometrySpatialField
    - heatmap 网格分面

RPT 与 LatLonPointSpatialField 分享共同的各种功能。有些在这里列出：  
  

    - 纬度/经度索引点数据；可能是多值的
    - 使用 geofilt 过滤器、bbox 过滤器和范围查询快速过滤（支持日期线交叉）
    - 通过 geodist 来 sort/boost  

    - 已知文本（WKT）形状语法（用于指定多边形和其他复杂形状）和 GeoJSON。除了索引和搜索之外，它还可以与 wt=geojson （GeoJSON Solr 响应写入器）和 [geo f=myfield]（geo Solr文档转换器）配合使用。

### RPT 的架构配置<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#schema-configuration-for-rpt"/>

要使用 RPT，必须在 schema.xml 中注册和配置字段类型。这个字段类型有很多选项。  

**name**
    
        字段类型的名称。  
    
**class**
    
        这应该是<code>solr.SpatialRecursivePrefixTreeFieldType</code>。但请注意，Lucene 空间模块除 RPT 之外还包含一些其他所谓的“空间策略”，特别是 TermQueryPT *，BBox，PointVector * 和 SerializedDV。Solr 需要一个字段类型才能并行使用它们。  
**spatialContextFactory**
    
        这是一个 Java 类名称，指向支持形状定义和解析的内部扩展点。如果您需要多边形支持，请将其设置为<code>JTS</code>- <code>org.locationtech.spatial4j.context.jts.JtsSpatialContextFactory</code>的别名；否则可以省略。请参阅下面有关 JTS 的重要信息。（注意：在 Solr 6之前，“org.locationtech.spatial4j”部分是“com.spatial4j.core”，并且以前没有方便的 JTS 别名）  
    
**geo**
    
        如果为<code>true</code>，则使用默认的纬度和经度坐标，数学模型通常是一个球体。如果为<code>false</code>，则在 2D 平面上使用欧氏/笛卡尔（Euclidean/Cartesian）几何的坐标将是一般的 X 和 Y。  
**format**
    
        定义要使用的形状语法/格式。默认为<code>WKT</code>但是<code>GeoJSON</code>是另一种流行的格式。Spatial4j 管理此功能并支持其他格式。如果一个给定的形状可以解析为“lat，lon”或“x y”，那么它总是被支持的。  
    
**distanceUnits**
    这用于指定在整个使用此字段时使用的距离测量单位。可以是<code>degrees</code>、<code>kilometers</code>或者<code>miles</code>。它适用于几乎所有的距离测量涉及的领域：<code>maxDistErr</code>，<code>distErr</code>，<code>d</code>，<code>geodist</code>和<code>score</code>当得分为<code>distance</code>，<code>area</code>或<code>area2d</code>。但是，它并不影响嵌入 WKT 字符串中的距离，（例如<code>BUFFER(POINT(200 10),0.2)</code>）仍然是度数。  
  

        
            <code>distanceUnits</code>默认为<code>kilometers</code>如果<code>geo</code>是 TRUE；或<code>degrees</code>如果<code>geo</code>是<code>false</code>。  
          
        
            <code>distanceUnits</code>替换<code>units</code>属性；现在已经被弃用，并且与这个属性互斥。  
          
    
**distErrPct**
    更大的<code>distErrPct</code>值将会使查询更快但不太准确。在查询时，这可以在查询语法中被覆盖，例如<code>0.0</code>，以便不近似于搜索形状。RPT 字段的默认值是<code>0.025</code>。  
  

        
            对于 RPTWithGeometrySpatialField （见下文），在序列化的几何中总是有完全的精度，所以这并不能控制精度，因为它控制了索引应该有多大的权衡。distErrPct 默认为该字段的0.15。  
**maxDistErr**
    
        定义索引数据所需的最高详细级别。如果留空，默认值是一米 - 仅略小于0.000009度。这个设置在内部用来计算一个合适的 maxLevels（见下文）。  
    
**worldBounds**
    
        定义 x 和 y 的有效数值范围，格式为<code>ENVELOPE(minX, maxX, maxY, minY)</code>。如果<code>geo="true"</code>，则假定标准的经纬度世界的边界。如果<code>geo=false</code>，您应该定义您的边界。  
**distCalculator**
    
        定义距离计算算法。如果<code>geo=true</code>，则“haversine” 是默认的。如果<code>geo=false</code>，则“cartesian”将是默认的。其他可能的值是“lawOfCosines”，“vincentySphere”和“cartesian^2”。  
    
**prefixTree**
    
        定义空间网格实现。由于 PrefixTree（如 RecursivePrefixTree）将世界映射为一个网格，因此每个网格单元将被分解到另一个位于下一级的网格单元。  
        
            如果<code>geo=true</code>那么默认的前缀树是<code>geohash</code>，否则是<code>quad</code>。Geohash 每个级别有 32 个子级，quad 有4个。Geohash 只能用于<code>geo=true</code>严格的地理空间。  
          
        
            第三个选择是<code>packedQuad</code>，一般来说<code>quad</code>，如果有很多层次（大概20个或更多层次），效率就会更高。  
          
    
**maxLevels**
    
        设置索引数据的最大网格深度。相反，通过指定<code>maxDistErr</code>来计算适当的 maxLevels 通常更直观。  


还有其他的一些选项：normWrapLongitude、datelineRule、validationRule、autoIndex、allowMultiOverlap、precisionModel。有关更多信息，请参阅下面有关 spatialContextFactory 实现的说明，尤其是指向基于 JTS 的链接。  
  

### JTS 和多边形<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#jts-and-polygons"/>

如上所示，spatialContextFactory 必须设置为多边形支持的 JTS，包括多边形。  
  
所有其他形状 (包括偶数行字符串) 都不受 JTS 支持。JTS 代表 JTS Topology Suite，由于其 LGPL 许可证，不包含 Solr。您必须下载（JAR 文件），并将它放到 Solr 特殊的地理位置内部：SOLR_INSTALL/server/solr-webapp/webapp/WEB-INF/lib/。您可以随时在此下载：https：
    //repo1.maven.org/maven2/com/vividsolutions/jts-core/。如果放置在其他更典型的 Solr 库目录中，则不起作用。  
激活时，还有其他配置属性可用；对于 Javadoc，请参阅 org.locationtech.spatial4j.context.jts.JtsSpatialContextFactory，并记住也要查看超类的选项。一个选项，尤其是您应该最有可能启用 autoIndex（即使用 JTS 的 PreparedGeometry），因为它已被证明是非平凡多边形的主要性能提升。  
```
&lt;fieldType name="location_rpt"   class="solr.SpatialRecursivePrefixTreeFieldType"
               spatialContextFactory="org.locationtech.spatial4j.context.jts.JtsSpatialContextFactory"
               autoIndex="true"
               validationRule="repairBuffer0"
               distErrPct="0.025"
               maxDistErr="0.001"
               distanceUnits="kilometers" /&gt;
```
一旦字段类型被定义，定义一个使用它的字段。  
下面是一个可以是 solr 的字段 "geo" 的多边形查询示例，SpatialRecursivePrefixTreeFieldType 或 RptWithGeometrySpatialField：  
```
&amp;q=*:*&amp;fq={!field f=geo}Intersects(POLYGON((-10 30, -40 40, -10 -20, 40 20, 0 0, -10 30)))
```
在搜索谓词后面的括号内是形状定义。该形状的格式由字段类型的“format”属性管理，默认为 WKT。如果您喜欢 GeoJSON，则可以指定。  
  
在这个参考指南和 Spatila4j's 文档之外, 还有一些详细信息保留在 http://wiki.apache.org/solr/SolrAdaptersForLuceneSpatial4 的 Solr Wiki 上。  
  

### RptWithGeometrySpatialField 字段类型<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#rptwithgeometryspatialfield"/>

RptWithGeometrySpatialField 字段类型是由 SpatialRecursivePrefixTreeFieldType 衍生的，它还将原始几何存储在 Lucene DocValues 内部，它用来实现精确的搜索。它也可以用于索引点字段。Intersects 谓词（默认值）特别快，因为许多搜索结果可以作为精确的命中返回而不需要几何检查。除了默认 distErrPct 值为 0.15（高于0.025）外，此字段类型的配置与 RPT类似，因为网格方块纯粹是为了性能，而不是从根本上表示形状。  
  
可以在 solrconfig.xml 定义一个可选的内存中缓存，当数据倾向于具有多个顶点的形状时，应进行此操作。假设您将字段命名为“geom”，则可以通过添加以下内容在 solrconfig.xml 中配置可选缓存 - 注意缓存名称的后缀：  
```
&lt;cache name="perSegSpatialFieldCache_geom"
           class="solr.LRUCache"
           size="256"
           initialSize="0"
           autowarmCount="100%"
           regenerator="solr.NoOpRegenerator"/&gt;
```
使用此字段类型时，您可能不希望将该字段标记为已存储，因为它与 DocValues 数据是多余的，并且由于格式化（无论是 WKT 还是 GeoJSON），它肯定更大。要从 DocValue 检索搜索结果中的空间数据，请使用[geo]转换 -  [转换结果文档](https://www.w3cschool.cn/solr_doc/solr_doc-mu4f2gta.html)。  
  

### Heatmap Faceting<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#heatmap-faceting"/>

RPT 字段支持为每个网格单元中具有空间数据的文档生成一个二维的分面计数网格。对于高细节网格，这可以用来绘制点，而对于较小的细节，它可以用于生成 heatmap。网格单元根据 RPT的配置在索引时间确定。在分面计数时，遍历感兴趣区域中的索引单元格，并且对应于每个单元格的计数器网格递增。Solr 可以在整数的直接 2D 数组中或在 PNG 中返回数据，也可以在较大数据集压缩的 PNG 中返回数据，但必须解码。  
  
Solr 的分面功能可以访问 heatmap 功能。作为分面的一部分，它支持 key 本地参数以及不包括标记的过滤器查询，就像其他类型的分面一样。这允许在具有不同过滤器的同一字段上返回多个 heatmap。  

**facet**
    
        设置<code>true</code>为启用刻面。  
    
**facet.heatmap**
    
        RPT 类型的字段名称。  
    
**facet.heatmap.geom**
    
        计算 heatmap 的区域，使用矩形范围语法或 WKT 指定。它默认为世界。例如：<code>["-180 -90" TO "180 90"]</code>。  
    
**facet.heatmap.gridLevel**
    
        一个特定的网格层次，决定每个网格单元的大小。默认为通过<code>distErrPct</code>（或<code>distErr</code>）计算。  
    
**facet.heatmap.distErrPct**
    
        用于计算 gridLevel 的几何大小的一小部分。默认为 0.15。它的计算方式与 RPT 中一个类似命名的参数相同。  
    
**facet.heatmap.distErr**
    
        用于间接选取网格级别的单元格错误距离。它的计算方式与 RPT 中一个类似命名的参数相同。  
    
**facet.heatmap.format**
    
        格式，可以是<code>ints2D</code>（默认）或<code>png</code>。  
    

Tip：您将试验不同的 distErrPct 值 (可能是 0.10-0.20) 与各种输入几何，直到默认大小为您要找的值。它的计算方法的具体细节并不重要。对于在点绘制中使用的 high-detail 网格 (每像素松散一个单元格)，将 distErr 设置为几个像素的十进制数，或者显示地图的数量。此外，您可能不希望使用 geohash-based 网格，因为网格级别之间的单元格方向 flip-flops 介于正方形和矩形之间。  
以下是 JSON 中的一些示例输出（为了简洁，插入了“...”）：  
```
{gridLevel=6,columns=64,rows=64,minX=-180.0,maxX=180.0,minY=-90.0,maxY=90.0,
counts_ints2D=[[0, 0, 2, 1, ....],[1, 1, 3, 2, ...],...]}
```
输出显示的 gridLevel 很有趣，因为它通常是从其他参数计算的。如果正在开发的接口允许明确的分辨率增加/减少特性，则后续请求可以明确指定 gridLevel。  
  
minX，maxX，minY，maxY 报告了这一地区的数量。这是 geom 在目标网格级输入的最小包围矩形。这可能会包装日期。columns 和 rows 值是输出矩形要平均划分的列和行数。注意：如果 geo = true，或者 geo = false，则单元格被赋值，因此单元格数据在十进制度数的坐标空间中时，不要均匀划分屏幕上投影的地图矩形以绘制这些矩形/点。这可以被安排成与屏幕上的地图相同，但不一定是这样。  
counts_ints2D 键有一个整数的二维数组。初始外部级别按行顺序（从上到下），内部数组是列（从左到右）。如果任何数组将全部为零，则返回 null，以提高效率。如果没有匹配的空间数据，则整个值为 null。  
如果 format=png，那么输出键是 counts_png。它是一个4字节的 PNG 的64位编码的字符串。PNG 逻辑上保存与 ints2D 格式相同的数据。请注意，alpha 通道字节会翻转，以便于查看PNG 用于诊断目的，否则计数将不得不超过2 ^ 24，然后才会变为不相符。因此，大于这个值的计数将变得不透明。  

## BBoxField<a href="http://lucene.apache.org/solr/guide/7_0/spatial-search.html#bboxfield"/>

BBoxField 对每个文档字段的单个矩形（边界框）进行索引，并支持通过边界框搜索。它支持大多数空间搜索谓词，它基于搜索矩形和索引矩形之间的重叠或区域，增强了相关性模式。它特别适用于它的关联模式。要在架构中配置它，使用像这样的配置：  
```
&lt;field name="bbox" type="bbox" /&gt;
&lt;fieldType name="bbox" class="solr.BBoxField"
           geo="true" distanceUnits="kilometers" numberType="pdouble" /&gt;
&lt;fieldType name="pdouble" class="solr.DoublePointField" docValues="true"/&gt;
```
BBoxField 实际上是基于由 numberType 引用的另一个字段类型的4个实例。它也使用布尔值来标记日期线十字。假设您想要使用相关性功能，则需要 docValues。一些属性与 RPT 字段（如geo、units、worldBounds 和 spatialContextFactory）是相同的，因为它们共享一些相同的空间基础结构。  
  
如果要为某个框进行索引，请将字段值添加到作为 WKT / CQL ENVELOPE 语法中的字符串的 bbox 字段中。例如：ENVELOPE(-10, 20, 15, 10)，这是minX，maxX，maxY，minY的顺序。参数排序是不直观的，但这是规范要求的。或者，您可以在 WKT 中提供一个矩形多边形（如果您设置了 set format="GeoJSON"，​​则为 GeoJSON）。  
如果要进行搜索，可以使用{!bbox}查询解析器或范围语法：[10,-10 TO 15,20]，或者使用带引导搜索谓词的圆括号中的 ENVELOPE 语法。后者是选择 Intersects 以外的谓词的唯一方法。例如：  
```
&amp;q={!field f=bbox}Contains(ENVELOPE(-10, 20, 15, 10))
```
现在，按相关性模式之一对结果进行排序，如下所示：  
```
&amp;q={!field f=bbox score=overlapRatio}Intersects(ENVELOPE(-10, 20, 15, 10))
```
该 score 本地参数可以是一个 overlapRatio，area 和 area2D。区域比分由文件区域使用 surface-of-a 球面（假定 geo=true）数学，area2D 使用简单的宽度*高度。overlapRatio 根据相对于文档区域和查询区域存在多少重叠来计算 [0-1] 距离分数。javadocs 的 BBoxOverlapRatioValueSource 有更多的信息的公式。还有一个额外的参数 queryTargetProportion 允许您将公式的查询端加权到公式的索引（目标）端。您也可以使用 &amp;debug=results 查看有用的分数计算信息。  
