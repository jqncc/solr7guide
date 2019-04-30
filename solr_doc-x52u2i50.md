## Solr身份验证和授权插件 
Solr拥有支持用户身份验证和授权的安全框架。这允许验证用户的身份并限制对Solr集群中的资源的访问。  
  
Solr包括一些随时可用的插件，并且可以使用下面描述的认证和授权框架来开发额外的插件。  
无论是在 SolrCloud 模式下还是在独立模式下运行， 所有的身份验证和授权插件都可以与 Solr 一起使用。所有身份验证和授权配置（包括用户和权限规则）都存储在名为“security.json”的文件中。在独立模式下使用Solr时，该文件必须位于$SOLR_HOME目录（通常为server/solr）中。当使用SolrCloud时，这个文件必须位于ZooKeeper中。  
以下部分描述如何启用security.json插件并将其放置在适合您的操作模式的位置。  
## 使用security.json启用插件
初始化任何类型的安全性插件所需的所有信息都存储在一个security.json文件中。该文件包含2个部分，其中每个部分用于身份验证和授权。  
  
示例security.json  
```
{
  "authentication" : {
    "class": "class.that.implements.authentication"
  },
  "authorization": {
    "class": "class.that.implements.authorization"
  }
}
```
在Solr实例出现之前，该/security.json文件需要位于适当的位置，所以Solr在启用安全插件的情况下启动。有关如何执行此操作的信息，请参阅下面的 "使用Solr中的security.json" 一节。  
  
根据所使用的插件，其他信息将被存储在security.json，例如用户信息或创建角色和权限的规则。这些信息是通过Solr提供的每个插件的API添加的，或者在自定义插件的情况下，由您设计的方法添加。  
这是一个更详细的security.json例子。在这里，启用了基本身份验证和基于规则的授权插件，并添加了一些数据：  
```
{
"authentication":{
   "class":"solr.BasicAuthPlugin",
   "credentials":{"solr":"IV0EHq1OnNrj6gvRCwvFwTrZ1+z1oBbnQdiVC3otuq0= Ndd7LKvVBAaZIF0QAVi1ekCfAJXr1GGfLtRUXhgrF8c="}
},
"authorization":{
   "class":"solr.RuleBasedAuthorizationPlugin",
   "permissions":[{"name":"security-edit",
      "role":"admin"}],
   "user-role":{"solr":"admin"}
}}
```
## 在Solr中使用security.json

### 在SolrCloud模式中使用
在配置 Solr 使用身份验证或授权插件时，您需要将security.json文件上传到ZooKeeper。以下的命令会在上传文件时写入文件 - 您也可以上传已经在本地创建的文件。  
```
&gt;server/scripts/cloud-scripts/zkcli.sh -zkhost localhost:2181 -cmd put /security.json
  '{"authentication": {"class": "org.apache.solr.security.KerberosPlugin"}}'
```
请注意，这个例子定义了用于身份验证的 KerberosPlugin。您将需要根据您所使用的插件修改本部分。  
  
这个例子还定义了命令行上的security.json，但是您也可以在本地定义一个文件并把它上传到ZooKeeper。  
注意：根据您使用的身份验证和授权插件，您可能有存储在<code>security.json</code>中的用户信息。如果是这样，我们强烈建议您在ZooKeeper节点中实施访问控制。有关如何启用此功能的信息, 可在 "ZooKeeper访问控制" 一节中获得。  
一旦security.json上传到ZooKeeper，您应该使用适当的API来更新您使用的插件。您可以手动对其进行编辑，但必须注意删除任何版本数据，以便在所有ZooKeeper节点上正确更新。版本数据位于security.json文件的末尾，并以字母“v”后面跟着一个数字，例如：{"v":138}。  
  
### 在独立模式下使用
在独立模式下运行Solr时，您需要创建security.json文件并将其放在您安装的 $SOLR_HOME 目录中（这与您通常所在的solr.xml的位置相同并且通常为server/solr）。  
  
如果您正在使用Legacy Scaling 和 Distribution，则需要在集群的每个节点上放置security.json。  
您可以使用身份验证和授权API，但是如果您使用旧版缩放模型，则需要分别在每个节点上创建相同的API请求。如果您愿意，也可以手动编辑security.json。  
## 身份验证插件
身份验证插件通过认证传入的请求进行身份验证来帮助保护 Solr 的端点。自定义插件可以通过扩展AuthenticationPlugin类来实现。  
  
身份验证插件由两部分组成：  
1 <li>服务器端组件，它使用插件中定义的机制（例如Kerberos、基本身份验证或其他方法）来拦截和验证传入的对 Solr 的请求。</li>2 <li>客户端组件，即一个扩展HttpClientConfigurer，它使SolrJ客户端能够使用服务器可以理解的身份认证机制向安全的Solr实例发出请求。</li>
### 启用插件
- 请在/security.json中指定身份验证插件，如以下示例所示：
```
{
  "authentication": {
    "class": "class.that.implements.authentication",
    "other_data" : "..."}
}
```
- security.json认证块中的所有内容将在初始化期间作为映射传递给插件。
- 身份验证插件也可以通过在启动期间传入-DauthenticationPlugin=&lt;plugin class name&gt;来与独立的Solr实例一起使用。
### 可用的身份验证插件
Solr具有以下验证插件的实现：  
- Kerberos身份验证插件
- 基本身份验证插件
- Hadoop身份验证插件
## 授权
通过扩展AuthorizationPlugin接口，可以为Solr编写授权插件。  
### 加载自定义插件
- 确保插件实现在类路径中。
- 然后可以通过在security.json中指定相同的方法来初始化插件，方式如下：
```
{
  "authorization": {
    "class": "org.apache.solr.security.MockAuthorizationPlugin",
    "other_data" : "..."}
}
```
security.json中的authorization块中的所有内容将在初始化期间作为映射传递给插件。  
  
注意：授权插件仅在SolrCloud模式下受支持。另外，重新加载插件还不受支持，需要重启 Solr 安装 (也就是说，应该重新启动 JVM，而不是简单地重新加载内核)。  
### Solr有一个授权插件的实现：可用的授权插件
- 基于规则的授权插件
## 确保节点间请求的安全
有很多来自Solr节点本身的请求。例如，监督节点、恢复线程等的请求。每个Authentication（身份验证）插件声明它是否能够保证节点间的请求。如果没有，Solr将回退到使用特殊的节点间身份验证机制，其中每个Solr节点是超级用户，并被其他Solr节点完全信任，如下所述。  
  
### PKIAuthenticationPlugin
当在两个Solr节点之间发生任何请求时，使用PKIAuthenticationPlugin，并且配置的Authentication插件不希望处理节点间安全性。  
对于每个传出的请求，PKIAuthenticationPlugin添加一个特殊的标题'SolrAuth'，其中包含使用该节点的私钥加密的时间戳和主体。公钥通过API公开，任何节点只要需要就可以读取。任何获取该头部请求的节点都会从发送方获取公钥并解密信息。如果它能够解密数据，则请求受信任。如果时间戳超过5秒，则它无效。这假定群集中不同节点的时钟是同步的。  
超时时间可通过称为 pkiauth.ttl 的系统属性进行配置。例如，如果您希望将生存时间提高到10秒（10000毫秒），请使用属性'-Dpkiauth.ttl=10000'启动每个节点。  
