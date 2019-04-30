## 在SolrCloud模式下添加自定义插件 
<div class="content-intro view-box ">在SolrCloud模式下，自定义插件需要在群集的所有节点之间共享。
      
  
当在SolrCloud模式下运行Solr并且想要使用自定义代码（例如自定义分析器、标记器、查询解析器和其他插件）时，将jar添加到群集中所有节点上的类路径可能非常麻烦。使用 Blob Store API和Config API的特殊命令，您可以将jar上传到一个特定的系统级集合，并在运行时从它们中动态加载插件，而无需重新启动任何节点。  
默认情况下禁用此功能，除了通过在 SolrCloud 模式下运行来要求该 solr 之外，默认情况下此功能也是禁用的，除非所有Solr节点在启动时都使用该<code>-Denable.runtime.lib=true</code>选项运行。在启用此功能之前，用户应仔细考虑以下"安全运行库"部分中讨论的问题  

## 上传Jar文件

第一步是使用Blob Store API来上传你的jar文件。这将把您的罐子.system集合中，并将它们分布在您的SolrCloud节点上。这些jar被添加到一个单独的类加载器，并且只可由配置为 runtimeLib = true 的组件访问。这些组件被惰性加载，因为当一个特定的核心被加载时，.system集合可能不会被加载。  
  

## 配置使用Jars作为运行时库的API命令

运行时库功能为Config API使用一组特殊的命令来将blob存储中当前可用的jar文件添加、更新或删除到运行时库列表中。  
  
以下命令用于管理运行时库：  

    - add-runtimelib
    - update-runtimelib
    - delete-runtimelib
```
curl http://localhost:8983/solr/techproducts/config -H 'Content-type:application/json' -d '{
   "add-runtimelib": { "name":"jarblobname", "version":2 },
   "update-runtimelib": { "name":"jarblobname", "version":3 },
   "delete-runtimelib": "jarblobname"
}'
```
要使用的名称是您将jar上传到blob存储区时指定的blob的名称。您还应该包含您要使用的blob存储区中找到的jar版本。这些细节信息被添加到configoverlay.json中。  
  
默认 SolrResourceLoader 对已定义为运行库的 jar 没有可见性。有一个类加载器可以访问这些只能提供给特殊注释的组件的jar。  
每个可插入组件都可以有一个称为runtimeLib=true的可选的额外属性，这意味着组件在核心加载时不会被加载。相反，他们将被按需加载。如果组件加载时所有依赖的jar都不可用，则会抛出错误。  
这个例子显示了使用已经加载到Blob存储的jar创建一个ValueSourceParser。  
```
curl http://localhost:8983/solr/techproducts/config -H 'Content-type:application/json' -d '{
  "create-valuesourceparser": {
    "name": "nvl",
    "runtimeLib": true,
    "class": "solr.org.apache.solr.search.function.NvlValueSourceParser,
    "nvlFloatValue": 0.0 }
}'
```

## 保护运行时库

此功能的一个缺点是可以用来把恶意的可执行代码加载到系统中。但是，可以通过使用 PKI 来限制系统只加载受信任的 jar，以验证加载到系统中的可执行文件是否可信。  
  
以下步骤将允许您启用此功能的安全性。这些指令假设您已经使用-Denable.runtime.lib=true启动了所有的Solr节点了。  

### 第1步：生成一个RSA私钥

第一步是生成一个RSA私钥。下面的示例使用512位密钥，但是您应该使用适合您需要的强度。  
```
$ openssl genrsa -out priv_key.pem 512
```

### 步骤2：输出公钥

密钥的公共部分应该以DER格式输出，以便Java可以读取它。  
```
$ openssl rsa -in priv_key.pem -pubout -outform DER -out pub_key.der
```

### 步骤3：将密钥加载到ZooKeeper

然后, 从步骤2输出的. der 文件应加载到一个节点/密钥/exe 下的管理员, 以便在每个节点中都可用。您可以将任意数量的公钥加载到该节点上, 并且全部都是有效的。如果从目录中删除了某个密钥, 则该密钥的签名将不再有效。因此, 在删除密钥之前, 请确保使用更新 runtimelib 命令更新运行库配置, 并使用有效的签名。  
在目前的时间, 你只能使用动物园管理员 zkCli (或 zkCli 在 Windows 上) 脚本来发出这些命令 (Solr 版本具有相同的名称, 但不相同)。如果你有自己的动物园管理员合奏已经运行, 你可以找到脚本在 $ZK _install/斌/zkCli (或 zkCli, 如果您正在使用的 Windows)。  
  
  
  
完成第二步后应该将从步骤2输出的.der文件加载到/keys/exe节点下的ZooKeeper中，以便在每个节点中都可用。您可以将任意数量的公钥加载到该节点，并且全部都是有效的。如果从目录中删除一个密钥，该密钥的签名将不再有效。因此，在删除密钥之前，请确保使用该update-runtimelib命令更新运行时库配置的有效签名。  
目前，您只能使用ZooKeeper zkCli.sh（或Windows中的zkCli.cmd）脚本来发出这些命令（Solr版本具有相同的名称，但不一样）。如果您已经拥有自己的ZooKeeper集成，则可以在其中找到该脚本$ZK_INSTALL/bin/zkCli.sh（或者zkCli.cmd，如果您使用的是Windows）。  
如果您正在运行的是包含在 Solr 中的嵌入式ZooKeeper，则您已经没有此脚本；为了使用它，您将需要从 http://zookeeper.apache.org/下载一个ZooKeeper v3.4.10的副本。不用担心配置下载，您只是想获取命令行实用程序脚本。当您启动脚本时，您将连接到嵌入式ZooKeeper。  
要加载密钥，您需要使用zkCli.sh连接到ZooKeeper，创建目录，然后创建密钥文件，如下例所示：  
```
# Connect to ZooKeeper
# Replace the server location below with the correct ZooKeeper connect string for your installation.
$ .bin/zkCli.sh -server localhost:9983
# After connection, you will interact with the ZK prompt.
# Create the directories
[zk: localhost:9983(CONNECTED) 5] create /keys
[zk: localhost:9983(CONNECTED) 5] create /keys/exe
# Now create the public key file in ZooKeeper
# The second path is the path to the .der file on your local machine
[zk: localhost:9983(CONNECTED) 5] create /keys/exe/pub_key.der /myLocal/pathTo/pub_key.der
```
在此之后，任何加载jar的尝试都将失败。您的所有 jar 必须与您的一个私钥签名，以便Solr信任它。在步骤4-6中概述了为 jar 签名和使用签名的过程。  
  

### 第4步：签署jar文件

接下来，您需要签署jar文件的sha1摘要并获取base64字符串。  
```
$ openssl dgst -sha1 -sign priv_key.pem myjar.jar | openssl enc -base64
```
此步骤的输出将是一个字符串，您将需要在下面的步骤6中将该jar添加到您的类路径中。  
  

### 第5步：将jar加载到Blob Store

使用Blob Store API将您的jar加载到Blob存储区。这一步不需要签名；您将需要在步骤6中的签名将其添加到您的类路径中。  
```
curl -X POST -H 'Content-Type: application/octet-stream' --data-binary @{filename}
http://localhost:8983/solr/.system/blob/{blobname}
```
您在此步骤中给出jar文件的blob名称将被用作下一步中的名称。  

### 第6步：将jar添加到Classpath

最后，使用Config API将jar添加到类路径，如上所述。在这一步中，您将需要提供您在步骤4中获得的jar的签名。  
```
curl http://localhost:8983/solr/techproducts/config -H 'Content-type:application/json'  -d '{
  "add-runtimelib": {
    "name":"blobname",
    "version":2,
    "sig":"mW1Gwtz2QazjfVdrLFHfbGwcr8xzFYgUOLu68LHqWRDvLG0uLcy1McQ+AzVmeZFBf1yLPDEHBWJb5KXr8bdbHN/
           PYgUB1nsr9pk4EFyD9KfJ8TqeH/ijQ9waa/vjqyiKEI9U550EtSzruLVZ32wJ7smvV0fj2YYhrUaaPzOn9g0=" }
}'
```
