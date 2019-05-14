# Solr Schema API

Schema API是为Core/Collection提供的对Solr Schema的读取和写入访问的http接口。所有Schema元素都可以被读取访问，Field、DynamicField、FieldType和CopyField这些元素通过API来添加、删除或修改。

>为什么不鼓励手动编辑托管架构？  
>在示例配置中managed-schema文件可能包括一个建议不要手动编辑文件的注释。在架构 API 存在之前, 此类编辑是对架构进行更改的唯一方法，用户可能会强烈的希望继续以这种方式进行更改。  
>之所以不鼓励这样做，是因为如果后面使用了API进行更改，则可能会之前手式编辑的修改可能会丢失，除非在使用API修改前重新加载了core或重启Solr。如果在手动编辑后总是重新加载或重新启动, 那么执行这些编辑就没有问题了。  

在使用API修改架构时，将自动进行核心重新加载，以便随后对其索引的文档立即进行更改。以前编入索引的文档不会自动更新 - 如果现有索引数据使用您更改的架构元素，则必须索引它们。

修改了某个字段的类型，可能需要重建所有索引数据，如果未重建索引，你可能无法访问索引文档。修改schema.xml(managed-schema)文件并不会影响已经索引了的文档，但想要应用schema.xml文件的修改则需要重建索引

更改后所做的查询和更新可能会遇到在更改之前不存在的错误。完全删除索引并重建它通常是解决此类错误的唯一选项。

API的基本地址为http://&lt;host&gt;: &lt;port&gt;/solr/&lt;collection_name&gt;。例如，如果您运行Solr的 "cloud" 示例 (通过下面显示的 bin/solr 命令), 创建 "gettingstarted" 集合, 则该集合的基本 URL（如本节中的所有示例URL）将是：http://localhost:8983/solr/gettingstarted。

```
bin/solr -e cloud -noprompt
```

## Schema API 入口点

- /schema：检索架构，或修改架构以添加、删除或替换字段、动态字段、复制字段或字段类型。
- /schema/fields：检索有关所有定义的字段或特定命名字段的信息。
- /schema/dynamicfields：检索有关所有动态字段规则或特定命名动态规则的信息。
- /schema/fieldtypes：检索有关所有字段类型或特定字段类型的信息。
- /schema/copyfields：检索有关复制字段的信息。
- /schema/name：检索模式名称。
- /schema/version：检索模式版本。
- /schema/uniquekey：检索已定义的 uniqueKey。
- /schema/similarity：检索全局相似性定义。
