## Solr身份验证 
<div class="content-intro view-box ">Solr 的 bin/solr 脚本允许启用或禁用基本身份验证，允许您从命令行配置身份验证。  
  
目前，该脚本只启用基本身份验证，并且只有在使用 SolrCloud 模式时才可用。  

## 启用基本身份验证

### <a href="http://lucene.apache.org/solr/guide/7_0/solr-control-script-reference.html#enabling-basic-authentication"/>

命令 bin/solr auth enable 配置 Solr 以在访问用户界面时使用基本认证，使用 bin/solr 和任何 API 请求。  
Tip：有关 Solr 身份验证插件的详细信息，请参见保护 Solr 部分。有关基本身份验证支持的更多信息，请参阅基本身份验证插件一节。  
bin/solr auth enable 命令对启用基本身份验证进行了几项更改：  

    - 创建一个 security.json 文件并将其上传到 ZooKeeper。该 security.json 文件将看起来类似于：
```
{
  "authentication":{
   "blockUnknown": false,
   "class":"solr.BasicAuthPlugin",
   "credentials":{"user":"vgGVo69YJeUg/O6AcFiowWsdyOUdqfQvOLsrpIPMCzk= 7iTnaKOWe+Uj5ZfGoKKK2G6hrcF10h6xezMQK+LBvpI="}
  },
  "authorization":{
   "class":"solr.RuleBasedAuthorizationPlugin",
   "permissions":[
 {"name":"security-edit", "role":"admin"},
 {"name":"collection-admin-edit", "role":"admin"},
 {"name":"core-admin-edit", "role":"admin"}
   ],
   "user-role":{"user":"admin"}
  }
}
```

    - 将两行添加到 bin/solr.in.sh 或 bin\solr.in.cmd 以设置身份验证类型，以及到 basicAuth.conf 的路径：
```
# The following lines added by ./solr for enabling BasicAuth
SOLR_AUTH_TYPE="basic"
SOLR_AUTHENTICATION_OPTS="-Dsolr.httpclient.config=/path/to/solr-6.6.0/server/solr/basicAuth.conf"
```

    - 创建文件 server/solr/basicAuth.conf 以存储与 bin/solr 命令一起使用的凭证信息。

该命令采用以下参数：  

**-credentials**
    
        用户名和密码格式为<code>username:password</code>（初始用户）。  
        
            如果您不想将用户名和密码作为参数传递给脚本，则可以选择该<code>-prompt</code>选项。无论是<code>-credentials</code>还是<code>-prompt</code>都必须指定。  
          
    
**-prompt**
        如果提示是首选，请将 true 作为参数传递，请求脚本提示用户输入用户名和密码。  
        
            无论是<code>-credentials</code>还是<code>-prompt</code> 必须指定。  
          
    
**-blockUnknown**
    
        如果为 true，则阻止所有未经身份验证的用户访问 Solr。该默认值为 false，这意味着未经身份验证的用户仍然可以访问 Solr。  
    
**-updateIncludeFileOnly**
    
        如果为 true，则只有<code>bin/solr.in.sh</code>或<code>bin\solr.in.cmd</code>将更新设置，并且<code>security.json</code>不会创建。  
    
**-z**
    
        定义 ZooKeeper 连接字符串。如果要在所有 Solr 节点启动之前启用身份验证，这非常有用。  
    
**-d**
    
        定义 Solr 服务器目录，默认情况下为<code>$SOLR_HOME/server</code>。需要重写默认值并不常见，只有在自定义<code>$SOLR_HOME</code>目录路径时才需要。  
    
**-s**
    
        定义<code>solr.solr.home</code>默认的位置<code>server/solr</code>。如果在同一台主机上有多个 Solr 实例，或者您已经自定义了<code>$SOLR_HOME</code>目录路径，则可能需要定义这个实例。  
    


## 禁用基本身份验证

### <a href="http://lucene.apache.org/solr/guide/7_0/solr-control-script-reference.html#disabling-basic-authentication"/>

您可以使用 bin/solr auth disable 禁用基本身份验证。  
如果该-updateIncludeFileOnly 选项设置为 true，则只有 bin/solr.in.sh 或 bin\solr.in.cmd 将更新设置，并且 security.json 不会被删除。  
如果 -updateIncludeFileOnly 选项设置为假，则在 bin/solr.in.sh 或 bin\solr.in.cmd 中的设置将被更新，并且 security.json 将被删除。但是，该 basicAuth.conf 文件不会被任何选项删除。  
