# 查询组件设置

本节中的设置将影响Solr处理和响应查询的方式。

这些设置都是在 solrconfig. xml 中的 &lt;query&gt; 元素的子元素中配置的。

```
<query>
  ...
</query>
```

## 缓存定义

Solr缓存与索引搜索器的特定实例关联，索引的特定视图在该搜索器的生命周期内不发生变化。只要索引搜索器正在被使用，其缓存中的任何项目都将是有效的并可供重用。Solr中的缓存与许多其他应用程序中的缓存不同，因为缓存的Solr对象在一段时间间隔后不会过期；相反，它们在索引检索器的整个生命周期内仍然有效。

当新的搜索器被打开时，当前的搜索器将继续服务请求，而新的搜索者则auto-warms(预热)其缓存。新的搜索器使用当前搜索的缓存预先填充自己的缓存。当新的搜索器准备好时，它被注册为当前的搜索器，并开始处理所有新的搜索请求。一旦完成了所有的请求，旧的搜索器将被关闭。

在Solr中，有三个缓存实现：solr.search.LRUCache，solr.search.FastLRUCache和solr.search.LFUCache。

LRU代表最近最少的使用。当LRU缓存填满时，最后访问时间是最早的数据被删除。最终的结果是被频繁访问的数据倾向于停留在缓存中，而那些不经常访问的数据清除出缓存，并且如果需要的话将会从索引中重新获取。

FastLRUCache被设计成无锁的，所以它非常适合于在请求中多次触发的缓存

LRUCache和FastLRUCache都使用 auto-warm 计数来支持整数和百分比，当 warm 发生时，它会相对于缓存的当前大小进行评估。

LFUCache指最不常用的缓存。与LRU策略类似，不同之处在于当缓存填满时，最少使用的数据将被删除。

Solr Admin UI中的Statistics页面将显示有关所有活动缓存的性能的信息。这些信息可以帮助您针对特定应用程序适当调整各种缓存的大小。当搜索器终止时，其缓存使用情况的摘要也被写入日志。

每个缓存都有设置来定义它的初始大小（initialSize），最大大小（size）和项目的数量，以用于warm（autowarmCount）。LRU和FastLRU缓存实现可以采取一个百分比，而不是autowarmCount的绝对值。

FastLRUCache和LFUCache支持showItems属性。这是要在缓存的统计信息页面中显示的缓存项目的数量。这是为了调试。
下面介绍每个缓存的详细信息。

### filterCache

这个缓存被SolrIndexSearcher用来过滤生成DocSets,当一个IndexSearcher实例打开时,它的缓存会被填充或者从一个旧的IndexSearcher实例的缓存里提前预热

filterCache常用于缓存每个fq搜索参数的结果，尽管还有其他一些情况。使用相同参数过滤器查询的后续查询会导致缓存命中并快速返回结果。请参阅搜索有关fq参数的详细讨论。使用此缓存的另一个Solr功能是filter(…​)默认Lucene查询解析器中的语法。

当配置参数facet.method设置为时，Solr还将此缓存用于分面fc。有关分面的讨论，请参阅搜索

```xml
<filterCache class="solr.LRUCache"
             size="512"
             initialSize="512"
             autowarmCount="128"/>
```

### 查询结果缓存queryResultCache

该缓存保存以前搜索的结果：基于查询、排序和所请求的文档范围的文档ID（DocList）的有序列表。

queryResultCache属性maxRamMB（单位MB,可选）设置，限制使用的最大RAM。当缓存增长超过此大小时，最早访问的查询将被逐出，直到缓存的堆使用率降至指定的限制以下。

```xml
<queryResultCache class="solr.LRUCache"
                  size="512"
                  initialSize="512"
                  autowarmCount="128"
                  maxRamMB="1000" />
```

### documentCache

这个缓存包含Lucene文档对象（每个文档的存储字段）。由于Lucene的内部文档ID是瞬态的，所以这个缓存不是自动预热的。为了确保Solr在请求期间不需要重新获取文档，documentCache应该始终大于max_resultstimes乘以max_concurrent_queries。您存储在文档中的字段越多，此缓存的内存使用率就越高。

```xml
<documentCache class="solr.LRUCache"
               size="512"
               initialSize="512"
               autowarmCount="0"/>
```

### 用户自定义的缓存

用可实现Solr的CacheRegenerator接口来自定义缓存。自定义可以通过SolrIndexSearcher的方法getCache()，cacheLookup()来访问,通过cacheInsert()插入缓存。

```xml
<cache name="myUserCache" class="solr.LRUCache"
                          size="4096"
                          initialSize="1024"
                          autowarmCount="1024"
                          regenerator="org.mycompany.mypackage.MyRegenerator" />
```

如果要auto-warming缓存，请包含一个regenerator属性的具有实现solr.search.CacheRegenerator类的完全限定名称。您也可以使用NoOpRegenerator，它只是简单地重新填充旧项目的缓存。使用regenerator参数来定义它，比如：regenerator =“solr.NoOpRegenerator”。

## 查询大小和 Warm

### maxBooleanClauses

maxBooleanClauses配置选项设置每个BooleanQuery最多支持的条件个数,如果超出此限制，则会引发异常。

```xml
<maxBooleanClauses>1024</maxBooleanClauses>
```

>此选项修改影响所有Solr核心的全局属性。如果多个solrconfig.xml文件在这个属性上不一致，则任何时候的值都将基于初始化的最后一个Solr核心。

### enableLazyFieldLoading

如果此参数设置为true，则将根据需要延迟加载未直接请求的字段。如果最常见的查询只需要一小部分字段，这可以提高性能，特别是如果不经常访问的字段很大。

```xml
<enableLazyFieldLoading>true</enableLazyFieldLoading>
```

### useFilterForSortedQuery

如果查询请求没有使用“score”排序，是否使用filter来替换Query,对于大多数情况这个配置没有用，除非频繁使用不同的排序规则来频繁重复执行相同的查询请求并且不要求返回文档评分score,这个配置才有用。

```xml
<useFilterForSortedQuery>true</useFilterForSortedQuery>
```

### queryResultWindowSize

使用查询结果集缓存的一个优化.当执行一个查询请求时,会收集到返回document的id集合,例如，如果响应于特定查询的搜索请求文档10到19，并且queryWindowSize是50，则文档0到49将被高速缓存。

```xml
<queryResultWindowSize>20</queryResultWindowSize>
```

### queryResultMaxDocsCached

在查询结果缓存中能够缓存的document最大个数

```xml
<queryResultMaxDocsCached>200</queryResultMaxDocsCached>
```

### useColdSearcher

使用冷查询,此设置控制是否有当前没有注册的搜索器的搜索请求应等待新的搜索器warm（false）或立即执行（true）。当设置为false时，请求将被阻塞，直到搜索器已经完成缓存预热为止。

```xml
<useColdSearcher>false</useColdSearcher>
```

### maxWarmingSearchers

此参数设置在后台并发进行缓存预热的IndexSearcher搜索器的最大个数。超过这个限制会引发错误。对于只读，值为2是合理的。

```xml
<maxWarmingSearchers>2</maxWarmingSearchers>
```

## 与查询相关的监听器

如缓存部分所述，新索引搜索器被缓存。可以使用监听器的触发器执行与查询有关的任务。最常见的用途是定义查询，以便在索引搜索器启动时进一步“预热”索引搜索器。这种方法的一个好处是，可以预先填充字段缓存以进行更快的排序。

良好的查询选择对于这种类型的监听器是关键的。最好选择最常见或最重的查询，不仅包括使用的关键字，还包括任何其他参数，如排序或过滤请求。

有两种类型的事件可以触发监听器。正在准备一个新的搜索时，会发生firstSearcher事件，但没有当前注册搜索来处理请求或获得从auto-warming的数据（例如，Solr上的启动）。每当有新的搜索器准备好时，就会触发一个newSearcher事件，并有一个当前的搜索器处理请求。
下面的（注释掉的）例子可以在Solr sample_techproducts_configs 配置集的solrconfig.xml的文件中找到，并且演示如何使用这个solr.QuerySenderListener类来warm一组显式查询：

```xml
<listener event="newSearcher" class="solr.QuerySenderListener">
  <arr name="queries">
  <!--
    <lst></lst><str name="q"></str>solr</str><str name="sort"></str>price asc</str></lst>
    <lst></lst><str name="q"></str>rocks</str><str name="sort"></str>weight asc</str></lst>
   -->
  </arr>
</listener>
<listener event="firstSearcher" class="solr.QuerySenderListener">
  <arr name="queries">
    <lst><str name="q">static firstSearcher warming in solrconfig.xml</str></lst>
  </arr>
</listener>
```

上面的代码来自一个solrconfig . xml示例。
一个关键的最佳实践是在将应用程序应用到生产之前修改这些默认值，但是请注意：当示例查询在“newSearcher”部分被注释掉的时候，示例查询并没有被注释为“firstSearcher”事件。

如果与您的搜索应用程序无关，使用查询字符串“static firstSearcher在solrconfig.xml中warm”来auto-warming索引搜索器是毫无意义的。