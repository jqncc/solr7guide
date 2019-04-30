## Solr命令行实用程序 
<div class="content-intro view-box ">ZooKeeper的命令行界面（CLI）脚本可用于直接与存储在ZooKeeper中的Solr配置文件进行交互。
      
  
虽然Solr的管理用户界面包括专用于SolrCloud群集状态的页面，但它不允许您下载或修改相关的配置文件。  
有关使用管理UI界面的更多信息， 请参阅Cloud Screens部分。
      
  
在server/scripts/cloud-scripts中找到的ZooKeeper CLI脚本允许您将配置信息上传到ZooKeeper，方法与参数引用中的示例中所示相同。它还提供了一些其他命令，使您可以将集合集链接到集合、创建ZooKeeper路径或清除它们，并将配置从ZooKeeper下载到本地文件系统。
      
  
zkCli.sh脚本提供的许多功能也由Solr控制脚本提供，可能更为熟悉，因为启动脚本ZooKeeper维护命令与Unix命令非常相似。
      
  

## Solr的zkcli.sh 与 ZooKeeper的zkCli.sh 对比

由 Solr 提供的<code>zkcli.sh</code>与包含在ZooKeeper中的<code>zkcli.sh</code>是不一样的。  
ZooKeeper的<code>zkcli.sh</code>提供了一个完全通用的、应用程序不可知的 shell，用于操作ZooKeeper的数据。本节中讨论的Solr的<code>zkcli.sh</code>是特定于 solr 的，并且具有特定于处理ZooKeeper中的Solr数据的命令行参数。  

## 使用Solr的ZooKeeper CLI

使用该help选项从脚本本身获取可用命令的列表，如./server/scripts/cloud-scrips/zkcli.sh help。  
这两个zkcli.sh（Unix环境下）和zkcli.bat（用于Windows环境）支持以下命令行选项：  
- -cmd &lt;arg&gt;  
    
        要执行的CLI命令。该参数是<strong>强制性的</strong>。支持以下命令：  
        
            <ul>
                <li>
                    <code>bootstrap</code>
                      
                
                - 
                    <code>upconfig</code>
                      
                
                - 
                    <code>downconfig</code>
                      
                
                - 
                    <code>linkconfig</code>
                      
                
                - 
                    <code>makepath</code>
                      
                
                - 
                    <code>get</code> 和 <code>getfile</code>
                      
                
                - 
                    <code>put</code> 和 <code>putfile</code>
                      
                
                - 
                    <code>clear</code>
                      
                
                - 
                    <code>list</code>
                      
                
                - 
                    <code>ls</code>
                      
                
                - 
                    <code>clusterprop</code>
                      
                
            
          
    </li><li>-z 或者 -zkhost &lt;locations&gt;  
    
        ZooKeeper主机地址。所有CLI命令都<strong>必须</strong>使用此参数。  
    </li><li>-c 或者 -collection &lt;name&gt;  
    
        对于<code>linkconfig</code>：集合的名称。  
    </li><li>-d 或者 -confdir &lt;path&gt;  
    
        对于<code>upconfig</code>：配置文件的目录。对于downconfig：从ZooKeeper提取的文件的目的地  
    </li><li>-h 或者 -help  
    
        显示帮助文字。  
    </li><li>-n 或者 -confname &lt;arg&gt;  
    
        对于<code>upconfig</code>，<code>linkconfig</code>，<code>downconfig</code>：配置集的名称。  
    </li><li>-r 或者 -runzk &lt;port&gt;  
    
        通过传递Solr运行端口在内部运行ZooKeeper；仅适用于一台机器上的群集。  
    </li><li>-s 或者 -solrhome &lt;path&gt;  
   
        为<code>bootstrap</code>或使用时<code>-runzk</code>：<strong>强制</strong> solrhome位置。  
    </li><li>-name &lt;name&gt;  
    
        对于<code>clusterprop</code>：<strong>强制性的</strong>集群属性名称。  
    </li><li>-val &lt;value&gt;  
   
        对于<code>clusterprop</code>：集群属性值。如果未指定，则将使用<strong>null</strong>作为值。  
    </li>
</ul>
提示：短格式参数选项可指定为单个破折号 (如:-c mycollection)。可以使用单个破折号 (如:<code>-collection mycollection</code>) 或双破折号 (如: <code>--collection mycollection</code>) 来指定长形式参数选项。  

## ZooKeeper CLI示例

下面是一些使用zkcli.shCLI的例子，假设你已经启动了SolrCloud的例子（bin/solr -e cloud -noprompt）  
如果你是Windows系统，只需在这些示例中将zkcli.sh更换为zkcli.bat。  

### 上传配置目录

```
./server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:9983 -cmd upconfig -confname my_new_config -confdir server/solr/configsets/_default/conf
```

### 从现有的solr.home引导ZooKeeper

```
./server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:2181 -cmd bootstrap -solrhome /var/solr/data
```

### 带 chroot 的引导

在- zkhost参数中使用 boostrap 命令与zookeeper chroot，如-zkhost 127.0. 0.1: 2181/solr，会在上传配置之前自动创建chroot路径。  

### 将任意数据放入一个新的ZooKeeper文件

```
./server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:9983 -cmd put /my_zk_file.txt 'some data'
```

### 把一个本地文件放到一个新的ZooKeeper文件中

```
./server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:9983 -cmd putfile /my_zk_file.txt /tmp/my_local_file.txt
```

### 将一个集合链接到配置集

```
./server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:9983 -cmd linkconfig -collection gettingstarted -confname my_new_config
```

### 创建一个新的ZooKeeper路径

在首次启动集群之前，在ZooKeeper中创建一个chroot路径会很有用。  
```
./server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:2181 -cmd makepath /solr
```

### 设置一个集群属性

该命令将在clusterprops.json中添加或修改单个集群属性。使用这个命令，而不是通常的getfile - &gt; edit - &gt; putfile循环。
      
  
与集合 API 上的 CLUSTERPROP 命令不同，这个命令也不会要求运行Solr的集群。  
```
./server/scripts/cloud-scripts/zkcli.sh -zkhost 127.0.0.1:2181 -cmd clusterprop -name urlScheme -val https
```
