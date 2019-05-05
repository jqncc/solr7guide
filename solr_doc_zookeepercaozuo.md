# ZooKeeper操作

bin/solr 脚本允许某些操作影响 ZooKeeper。这些操作仅适用于 SolrCloud 模式。这些操作可以作为子命令使用，每个子命令都有自己的一组选项。
  
```
bin/solr zk [sub-command] [options]
bin/solr zk -help
```

>Tip：Solr 应该已经启动至少一次，然后发出这些命令来初始化 ZooKeeper 和 Solr 所期望的 znodes。一旦 ZooKeeper 被初始化，Solr 就不需要在任何节点上运行来使用这些命令。

## 上传配置集

使用 zk upconfig 命令可以将预先配置的组态集或自定义配置集上传到 ZooKeeper。  

## ZK 上传参数

以下所有的参数都是必需的。  
**-n &lt;name&gt;：**  
ZooKeeper 中配置集的名称。这个命令会把配置集上传给“配置” ZooKeeper 节点，给它指定名字。  
您可以通过 Cloud 屏幕在管理界面中看到所有上传的配置集。选择 Cloud -&gt; Tree -&gt; configs 来查看它们。  
如果指定了预先存在的配置集，它将在 ZooKeeper 中被覆盖。 例如：-n myconfig

**-d &lt;configset dir&gt;：**  
要上传的配置集的路径。它应该在它下面有一个“conf” 目录，该目录又包含 solrconfig.xml 等。  
如果只提供一个名字，则将检查$SOLR_HOME/server/solr/configsets的名称。可能会提供一个绝对路径。 例如：
-d directory_under_configsets  
-d /path/to/configset/source  

**-z &lt;zkHost&gt;：**  
ZooKeeper 连接字符串。如果在 ZK_HOST 定义了solr.in.sh或者solr.in.cmd，则它是不必要的。例如： -z 123.321.23.43:2181

完整参数的命令示例如下：

```
bin/solr zk upconfig -z 111.222.333.444:2181 -n mynewconfig -d /path/to/configset
```

>注意：更改配置时重新加载集合，此命令不会自动使更改生效！它只是将配置集上传到 ZooKeeper。您可以使用集合 API 的 RELOAD 命令重新加载使用此配置集的所有集合。

## 下载配置集

使用 zk downconfig 命令从 ZooKeeper 下载配置集到本地文件系统。  

### ZK下载参数

下面列出的所有参数是必需的。  
**-n &lt;name&gt;：**  
在 ZooKeeper 中设置的配置名称。管理界面 Cloud -&gt; Tree -&gt; configs 节点列出所有可用的配置集。例如：-n myconfig
  
**-d &lt;configset dir&gt;：**  
将下载的配置集写入到的路径。如果只提供一个名称，$SOLR_HOME/server/solr/configsets将是父级。绝对路径也可以被提供。  
无论哪种情况，目标上预先存在的配置将被覆盖！  
例如：  
        -d directory_under_configsets  
        -d /path/to/configset/destination
-z &lt;zkHost&gt;：  
ZooKeeper 连接字符串。如果在 ZK_HOST 中定义了solr.in.sh或者olr.in.cmd，则它是不必要的。  例如：-z 123.321.23.43:2181
  
完整参数例子：

```
bin/solr zk downconfig -z 111.222.333.444:2181 -n mynewconfig -d /path/to/configset
```

"最佳做法" 是将您的配置集以某种形式的版本控制作为 system-of-record。在这种情况下，很少使用 downconfig。  

## 在本地文件和 ZooKeeper znodes 之间复制

使用该 zk cp 命令在 ZooKeeper znodes 和本地驱动器之间传输文件和目录。该命令将从本地驱动器复制到 ZooKeeper，从 ZooKeeper 复制到本地驱动器或从ZooKeeper 复制到 ZooKeeper。  

### ZK 复制参数

-r：  
可选的。执行一个递归的副本。如果 &lt;src&gt; 没有指定 “-r”，那么该命令将失败。  
例如：-r
  
&lt;src&gt;：  
要从中复制的文件或路径。如果前缀为zk:则源被假定为 ZooKeeper。如果没有前缀或前缀是 "file:"，则为本地驱动器。至少有一个&lt;src&gt; 或者 &lt;dest&gt; 必须使用前缀'zk:'否则该命令将失败。  
例如：  
- zk:/configs/myconfigs/solrconfig.xml
- file:/Users/apache/configs/src
  
&lt;dest&gt;：  
要复制到的文件或路径。如果前缀为 zk: 则源被假定为 ZooKeeper。如果没有前缀或前缀是 file: 这是本地驱动器。  
至少有一个&lt;src&gt;或者&lt;dest&gt;
  
必须使用前缀zk:否则该命令将失败。如果&lt;dest&gt;以斜杠字符结尾，则会命名一个目录。  
例如：  
- zk:/configs/myconfigs/solrconfig.xml
- file:/Users/apache/configs/src
  
-z &lt;zkHost&gt;：  
ZooKeeper 连接字符串。如果在 ZK_HOST 中定义了solr.in.sh或者olr.in.cmd，则它是不必要的。  
例如：-z 123.321.23.43:2181
  
带参数的这个命令的例子是：  
从本地递归复制一个目录到 ZooKeeper：

```
bin/solr zk cp -r file:/apache/confgs/whatever/conf zk:/configs/myconf -z 111.222.333.444:2181
```

将一个文件从 ZooKeeper 复制到本地：

```
bin/solr zk cp zk:/configs/myconf/managed_schema /configs/myconf/managed_schema -z 111.222.333.444:2181
```

## 从ZooKeeper中删除一个znode

使用 zk rm 命令从 ZooKeeper 中删除一个 znode（和可选的所有子节点）  

### ZK删除参数

**-r：**  
可选的。做一个递归删除。如果指定了 &lt;-r&gt;，那么如果 &lt;path&gt; 有子级，命令将失败。  

**&lt;path&gt;：**  
要从 ZooKeeper 中删除的路径，无论是父节点还是子节点。  
安全检查有限，不能删除 “/” 或 “/ zookeeper” 节点。  
路径被假定为一个 ZooKeeper 节点，不需要zk:前缀。例如：  
/configs  
/configs/myconfigset  
/configs/myconfigset/solrconfig.xml

**-z &lt;zkHost&gt;：**  
ZooKeeper 连接字符串。如果在 ZK_HOST 中定义了solr.in.sh或者olr.in.cmd，则它是不必要的。  
例如：-z 123.321.23.43:2181,示例：  
```
bin/solr zk rm -r /configs
bin/solr zk rm /configs/myconfigset/schema.xml
```

## 将一个 ZooKeeper znode 移动到另一个（重命名）

使用该zk mv命令来移动（重命名）ZooKeeper znode  

### ZK 移动参数

&lt;src&gt;：  
要重命名的 znode。假设zk:前缀。  
例如：/configs/oldconfigset
  
&lt;dest&gt;：  
znode 的新名称。假设zk:前缀。 例如：/configs/newconfigset

-z &lt;zkHost&gt;：  
ZooKeeper 连接字符串。如果在 ZK_HOST 中定义了solr.in.sh或者olr.in.cmd，则它是不必要的。  
例如：-z 123.321.23.43:2181,示例：

```
bin/solr zk mv /configs/oldconfigset /configs/newconfigset
```

## 列出 ZooKeeper znode 的子项

使用 zk ls 命令查看 znode 的子项。  

### ZK 列表参数

-r （可选）。递归地列出一个 znode 的所有子代。  
+ 示例：-r  
&lt;path&gt;：  
ZooKeeper 列出的路径。  
例如：/collections/mycollection
  
-z &lt;zkHost&gt;：  
ZooKeeper 连接字符串。如果在 ZK_HOST 中定义了solr.in.sh或者olr.in.cmd，则它是不必要的。  
例如：-z 123.321.23.43:2181
  
带参数的这个命令的例子是：  
```
bin/solr zk ls -r /collections/mycollection
bin/solr zk ls /collections
```


## 创建一个 znode（支持 chroot）

使用 zk mkroot 命令来创建一个 znode。此命令的主要用途是支持 ZooKeeper 的 “chroot” 概念。但是，它也可以用来创建任意路径。  

### 创建 znode 参数

&lt;path&gt;：  
ZooKeeper 创建的路径。如有必要，将创建中间节点。即使未指定，也会采用前导斜杠。  
例如：/solr
  
-z &lt;zkHost&gt;：  
ZooKeeper 连接字符串。如果在 ZK_HOST 中定义了solr.in.sh或者olr.in.cmd，则它是不必要的。  
例如：-z 123.321.23.43:2181
  
这个命令的例子：

```
bin/solr zk mkroot /solr -z 123.321.23.43:2181
bin/solr zk mkroot /solr/production
```