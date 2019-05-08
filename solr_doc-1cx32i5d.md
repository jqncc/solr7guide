# Hadoop身份验证插件 

Hadoop身份验证插件使Solr能够使用Hadoop身份验证库来保护Solr节点。

此身份验证插件是一个精简的包装，它将所有功能委派给 Hadoop 身份验证库。库的所有配置参数都通过插件传递。  
这个插件在利用Hadoop身份验证库中的一组扩展功能或新近可用的功能方面特别有用。  
请注意，Solr使用的Hadoop库版本是定期升级的。虽然Solr将确保插件配置结构（即插件的参数名称）的稳定性和向后兼容性，但这些参数的值可能会根据Hadoop库的版本而改变。有关更多详细信息，请查看您的Solr安装所使用的版本的Hadoop文档。  
对于一些身份验证方案（例如Kerberos），Solr提供了身份验证插件的本地实现。如果您需要更稳定的设置，在配置、执行滚动升级的能力、向后兼容性等方面，您应该考虑使用这样的插件。请查看“身份验证和授权插件”一节，了解有关 Solr 的身份验证插件选项概述。  
有两个插件类：
      
  

    - HadoopAuthPlugin：这可以与独立的Solr以及具有PKI身份验证的 Solrcloud一起用于节点间通信。
    - ConfigurableInternodeAuthHadoopPlugin：这是HadoopAuthPlugin的扩展，它允许您配置节点间通信的身份验证方案。

提示：对于大多数 SolrCloud 或独立的 Solr 设置，HadoopAuthPlugin 应该足够了。  

## 插件配置

- class  
        应该是solr.HadoopAuthPlugin或者solr.ConfigurableInternodeAuthHadoopPlugin。该参数是必需的。  
- type  
    
        要配置的认证方案的类型。请参阅配置选项。该参数是必需的。
- sysPropPrefix  
        用于定义配置认证机制的Java系统属性的前缀。此属性是必需的。  
        
            Java系统属性的名称是通过将配置参数名称附加到此前缀值来定义的。例如，如果前缀是<code>solr</code>那么Java系统属性<code>solr.kerberos.principal</code>定义了配置参数的值<code>kerberos.principal</code>。  
    
- authConfigs  
    
        由type属性定义的认证方案所需的配置参数。此属性是必需的。有关更多详细信息，请参阅Hadoop配置选项。  
    
- defaultConfigs  
    
        <code>authConfigs</code>属性指定了配置参数的默认值。默认值被指定为键值对（即，<code>"property-name": "default_value"</code>）的集合。  
    
- enableDelegationToken  
    
        如果为<code>true</code>，代表令牌功能将被启用。  
    
- initKerberosZk  
   
        在连接到ZooKeeper之前启用kerberos的初始化（如果适用）。  
    
- proxyUserConfigs  
   
        为代理用户配置基础Hadoop认证机制。该配置表示为键值对（即，<code>"property-name": "default_value"</code>）的集合。  
    
- clientBuilderFactory  
    
        NO| <code>HttpClientBuilderFactory</code>用于Solr的内部通信的实现。只适用于<code>ConfigurableInternodeAuthHadoopPlugin</code>。  
    

## 示例配置

### 使用Hadoop身份验证插件的Kerberos身份验证

这个例子允许您将 Solr 配置为使用 kerberos 身份验证，类似于使用 kerberos 身份验证插件的方式。
      
  
在查阅Hadoop认证库的文档后，您可以使用solr.*前缀来提供每个主机配置参数。例如，Hadoop认证库需要一个参数kerberos.principal，它可以在启动 solr 节点时作为一个名为 solr.kerberos.principal 的系统属性提供。有关其他典型配置参数，请参阅“Kerberos身份验证插件”部分。  
请注意，这个例子使用ConfigurableInternodeAuthHadoopPlugin，因此您必须提供clientBuilderFactory实现。因此，所有的节点间通信都将使用Kerberos机制，而不是PKI身份验证。  
要设置此插件，请在security.json文件中使用以下内容。  
```
{
    "authentication": {
        "class": "solr.ConfigurableInternodeAuthHadoopPlugin",
        "sysPropPrefix": "solr.",
        "type": "kerberos",
        "clientBuilderFactory": "org.apache.solr.client.solrj.impl.Krb5HttpClientBuilder",
        "initKerberosZk": "true",
        "authConfigs": [
            "kerberos.principal",
            "kerberos.keytab",
            "kerberos.name.rules"
        ],
        "defaultConfigs": {
        }
    }
}
```

### 使用委托令牌的简单身份验证

与前面的示例类似，这是设置使用委托令牌的Solr群集的示例。请参考Hadoop认证库文档中的参数，或者参考Kerberos认证插件了解更多细节。请注意，此示例不使用Kerberos，并且对Solr发出的请求必须包含有效的委托令牌。
      
  
要设置此插件，请在security.json文件中使用以下内容。  
```
{
    "authentication": {
        "class": "solr.HadoopAuthPlugin",
        "sysPropPrefix": "solr.",
        "type": "simple",
        "enableDelegationToken":"true",
        "authConfigs": [
            "delegation-token.token-kind",
            "delegation-token.update-interval.sec",
            "delegation-token.max-lifetime.sec",
            "delegation-token.renewal-interval.sec",
            "delegation-token.removal-scan-interval.sec",
            "cookie.domain",
            "signer.secret.provider",
            "zk-dt-secret-manager.enable",
            "zk-dt-secret-manager.znodeWorkingPath",
            "signer.secret.provider.zookeeper.path"
        ],
        "defaultConfigs": {
            "delegation-token.token-kind": "solr-dt",
            "signer.secret.provider": "zookeeper",
            "zk-dt-secret-manager.enable": "true",
            "token.validity": "36000",
            "zk-dt-secret-manager.znodeWorkingPath": "solr/security/zkdtsm",
            "signer.secret.provider.zookeeper.path": "/token",
            "cookie.domain": "127.0.0.1"
        }
    }
}
```
