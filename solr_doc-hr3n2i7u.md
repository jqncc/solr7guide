## Kerberos身份验证插件 
<div class="content-intro view-box ">如果您使用Kerberos来保护您的网络环境，则可以使用Kerberos身份验证插件来保护Solr集群。  
  
这允许Solr使用Kerberos服务主体和keytab文件来认证ZooKeeper以及Solr群集的节点（如果适用）。Admin UI和所有客户端（如SolrJ）的用户在能够使用UI或向Solr发送请求之前还需要拥有一个有效的票证。  
在SolrCloud模式或独立模式下支持Kerberos身份验证插件。  
如果您将Solr与通过Kerberos保护的Hadoop集群一起使用，并打算将您的Solr索引存储在HDFS中，请参阅HDFS上运行Solr部分，以获取为此目的配置Solr的其他步骤。本页面上的说明仅适用于使用Kerberos保护Solr的情况。如果您只需要将索引存储在Kerberized HDFS系统中，请参阅上面的其他部分。  

## Solr如何与Kerberos一起使用

当设置Solr以使用Kerberos时，Solr将使用服务主体或在密钥分发中心（KDC）注册的Kerberos用户名进行配置，以对请求进行身份验证。这些配置定义了服务主体名称和包含凭证的 keytab 文件的位置。  
  

### security.json

Solr 身份验证模型使用名为security.json的文件。此文件的说明以及创建和维护的方式都包含在身份验证和授权插件部分。如果此文件是在 Solr 初始启动后创建的，则需要重新启动系统的每个节点。  

### 服务主体和Keytab文件

每个Solr节点必须具有在密钥分发中心（KDC）注册的服务主体。Kerberos插件使用SPNego来协商身份验证。  
  
使用HTTP/host1@YOUR-DOMAIN.ORG，作为服务主体的一个例子：  

    - HTTP 指示这个服务主体将用于身份验证的请求的类型。服务主体中的HTTP/是 SPNego 在通过 http 对 Solr 的请求时必须使用的。
    - host1 是托管Solr节点的计算机的主机名。
    - YOUR-DOMAIN.ORG 是组织范围内的Kerberos领域。

同一主机上的多个Solr节点可能具有相同的服务主体，因为主机名对于所有主机都是通用的。  
与服务主体一起，每个Solr节点都需要一个keytab文件，该文件应包含所使用的服务主体的凭证。keytab 文件包含加密的凭据，以便在从KDC获取Kerberos票据时支持无密码登录。对于每个Solr节点，keytab 文件应保存在安全的位置，而不是与群集的用户共享。  
由于Solr群集需要节点间通信，因此每个节点还必须能够向其他节点启用Kerberos启用的请求。默认情况下，Solr使用相同的服务主体和keytab作为节点间通信的“客户端主体”。您可以明确地配置一个不同的客户端主体，但不建议这样做，下面的示例中没有涉及。  

### Kerberized ZooKeeper

在设置Kerberos SolrCloud集群时，建议同时启用ZooKeeper的Kerberos安全性。  
  
在这样的设置中，使用ZooKeeper身份验证请求的客户端主体也可以用于节点间通信。由于Solr使用的Zookeeper客户端负责这个操作，因此不需要分别续订票证授予票证（TGT）。为了达到这个目的，一个JAAS配置（应用程序名称为Client）可用于Kerberos插件以及Zookeeper客户端。  
有关在Kerberos模式下启动ZooKeeper的示例，请参见下面的ZooKeeper配置部分。  

### 浏览器配置

为了让您的浏览器在启用Kerberos身份验证后访问Solr管理用户界面，它必须能够与Kerberos身份验证器服务进行协商以允许您访问。每个浏览器都支持这种方式，有些（如Chrome）完全不支持。如果在启用Kerberos身份验证后尝试访问Solr管理用户界面时看到401错误，则可能是您的浏览器未正确配置，无法知道如何或在何处协商身份验证请求。  
  
有关如何设置浏览器的详细信息超出了本文档的范围。有关如何配置浏览器的详细信息，请与系统管理员联系以了解 Kerberos。  

## Kerberos身份验证配置

请咨询您的 Kerberos 管理员：在尝试将 Solr 配置为使用 kerberos 身份验证之前，请查看下面列出的每个步骤，并咨询您的本地Kerberos管理员的每个详细信息，以确保您知道每个参数的正确值。小错误可能会导致Solr无法启动或无法正常工作，并且非常难以诊断。  
Kerberos插件的配置有几个部分：  

    - 创建服务主体和 keytab 文件
    - ZooKeeper配置
    - 创建或更新 /security.json
    - 定义 jaas-client.conf
    - Solr启动参数

在之后的内容中我们会使用到这些步骤。  
```
注意：使用主机名；要使用主机名而不是 IP 地址, 请使用 bin/solr.in.sh 中的 SOLR_HOST 配置或在 solr 启动期间通过 -Dhost=&lt;hostname&gt; 系统参数。本指南使用 IP 地址。如果指定了主机名，请根据需要将指南中的所有IP地址替换为Solr主机名。
```

### 获取服务主体和Keytabs

在配置Solr之前，请确保您拥有KDC服务器中每个Solr主机和ZooKeeper的Kerberos服务主体（如果ZooKeeper尚未配置），并生成一个keytab文件，如下所示。  
  
这个例子假定主机名是192.168.0.107，并且您的主目录是/home/foo/。这个例子应该根据您自己的环境进行修改。  
```
root@kdc:/# kadmin.local
Authenticating as principal foo/admin@EXAMPLE.COM with password.
kadmin.local:  addprinc HTTP/192.168.0.107
WARNING: no policy specified for HTTP/192.168.0.107@EXAMPLE.COM; defaulting to no policy
Enter password for principal "HTTP/192.168.0.107@EXAMPLE.COM":
Re-enter password for principal "HTTP/192.168.0.107@EXAMPLE.COM":
Principal "HTTP/192.168.0.107@EXAMPLE.COM" created.
kadmin.local:  ktadd -k /tmp/107.keytab HTTP/192.168.0.107
Entry for principal HTTP/192.168.0.107 with kvno 2, encryption type aes256-cts-hmac-sha1-96 added to keytab WRFILE:/tmp/107.keytab.
Entry for principal HTTP/192.168.0.107 with kvno 2, encryption type arcfour-hmac added to keytab WRFILE:/tmp/107.keytab.
Entry for principal HTTP/192.168.0.107 with kvno 2, encryption type des3-cbc-sha1 added to keytab WRFILE:/tmp/108.keytab.
Entry for principal HTTP/192.168.0.107 with kvno 2, encryption type des-cbc-crc added to keytab WRFILE:/tmp/107.keytab.
kadmin.local:  quit
```
将 keytab 文件从KDC服务器的/tmp/107.keytab位置复制到位于的Solr主机/keytabs/107.keytab。对每个Solr节点重复此步骤。  
  
您可能需要采取类似的步骤来创建ZooKeeper服务主体和 keytab（如果尚未设置）。在这种情况下，下面的例子显示了ZooKeeper的一个不同的服务主体，因此上述可能会以 zookeeper/host1 作为其中一个节点的服务主体来重复。  

### ZooKeeper配置

如果您正在使用的ZooKeeper已配置为使用 Kerberos，则可以跳过此处显示的与ZooKeeper相关的步骤。  
  
由于ZooKeeper管理SolrCloud集群中节点之间的通信，因此它还必须能够与集群中的每个节点进行身份验证。配置需要为ZooKeeper设置一个服务主体，定义一个JAAS配置文件并指示ZooKeeper使用这两个项目。  
第一步是在ZooKeeper的conf目录下创建一个java.env文件并添加以下内容，如下例所示：  
```
export JVMFLAGS="-Djava.security.auth.login.config=/etc/zookeeper/conf/jaas-client.conf"
```
JAAS配置文件应包含以下参数。务必根据需要更改principal和keyTab路径。该文件必须位于上述步骤中定义的路径中，指定文件名。  
```
Server {
 com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  keyTab="/keytabs/zkhost1.keytab"
  storeKey=true
  doNotPrompt=true
  useTicketCache=false
  debug=true
  principal="zookeeper/host1@EXAMPLE.COM";
};
```
最后，将以下行添加到ZooKeeper的配置文件zoo.cfg中：  
```
authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider
jaasLoginRenew=3600000
```
一旦所有部分都到位，使用以下参数指向JAAS配置文件来启动ZooKeeper：  
```
bin/zkServer.sh start -Djava.security.auth.login.config=/etc/zookeeper/conf/jaas-client.conf
```

### 创建security.json

创建security.json文件，在SolrCloud模式下，您可以通过在创建ZooKeeper时上传security.json来将Solr设置为使用Kerberos插件，如下所示：  
```
server/scripts/cloud-scripts/zkcli.sh -zkhost localhost:2181 -cmd put /security.json '{"authentication":{"class": "org.apache.solr.security.KerberosPlugin"}}'
```
如果您在独立模式下使用Solr，则需要创建该security.json文件并将其放入您的$SOLR_HOME目录中。  
  
有关如何在Solr中使用/security.json文件的更多详细信息，请参见“身份验证和授权插件”部分。  
如果在ZooKeeper中已经有了/security.json文件，请下载该文件，添加或修改身份验证部分，然后使用 Solr 中提供的命令行实用程序将其上传回ZooKeeper。  
  

### 定义一个JAAS配置文件

JAAS配置文件定义用于身份验证的属性，如服务主体和 keytab 文件的位置。还可以设置其他属性以确保票据缓存和其他功能。  
  
以下示例可以根据您的环境稍微复制和修改。文件的位置可以在服务器上的任何位置，但在启动Solr时会被引用，因此它必须在文件系统上是可读的。JAAS文件可能包含不同用户的多个部分，但每个部分必须具有唯一的名称，以便在每个应用程序中唯一地引用它。  
在下面的例子中，我们创建了一个 JAAS 配置文件，其名称和路径为/home/foo/jaas-client.conf。我们将在下一节定义Solr start 参数时使用这个名称和路径。请注意，这里的客户端主体与服务主体相同。这将用于验证节点间的请求和请求到ZooKeeper。请确保使用正确的主体主机名和 keyTab 文件路径。  
```
Client {
  com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  keyTab="/keytabs/107.keytab"
  storeKey=true
  useTicketCache=true
  debug=true
  principal="HTTP/192.168.0.107@EXAMPLE.COM";
};
```
这个文件的第一行定义了节名称，它将与下面定义的solr.kerberos.jaas.appname参数一起使用。  
  
我们关注的主要属性是keyTab和principal属性，但也有其他可能需要为您的环境。Krb5LoginModule 的 javadoc（正在使用的类在上面的第二行中被调用）提供了可用属性的一个很好的概要，但是对于在上面的例子中使用的参考来说，这里解释如下：  

    - useKeyTab：这个布尔属性定义了我们是否应该使用一个keytab文件（在这种情况下是true）。
    - keyTab：JAAS配置文件的此部分用于主体的keytab文件的位置和名称。路径应该用双引号括起来。
    - storeKey：这个布尔属性允许密钥存储在用户的私人凭证中。
    - useTicketCache：该布尔属性允许从票证缓存中获取票证。
    - debug：此布尔属性将输出调试消息，以帮助进行疑难解答。
    - principal：要使用的服务主体的名称。

### Solr启动参数

在启动Solr时，需要传递以下特定于主机的参数。这些参数可以通过bin/solr启动命令在命令行中传递（请参阅“Solr控制脚本参考”以了解如何传递系统参数的详细信息），也可以在bin/solr.in.sh或者bin/solr.in.cmd中根据您的操作系统进行相应的定义。  
  

**solr.kerberos.name.rules**
    
        用于将Kerberos主体映射到短名称。默认值是<code>DEFAULT</code>。名称规则的例子：<code>RULE:[1:$1@$0](.*EXAMPLE.COM)s/@.*//</code>。  
    
**solr.kerberos.cookie.domain**
    
        用于发出cookie，并应具有Solr节点的主机名。该参数是必需的。  
    
**solr.kerberos.cookie.portaware**
    
        设置为<code>true</code>时，Cookie根据主机和端口进行区分，而标准Cookie不是端口可识别的。如果在同一个主机上托管多个Solr节点，应该设置此项。默认是<code>false</code>。  
    
**solr.kerberos.principal**
    
        服务主体。该参数是必需的。  
    
**solr.kerberos.keytab**
    
        包含服务主体凭证的Keytab文件路径。该参数是必需的。  
    
**solr.kerberos.jaas.appname**
    
        节点间通信所需的JAAS配置文件中的应用程序名称（节名称）。默认值是<code>Client</code>，也用于ZooKeeper身份验证。如果ZooKeeper和Solr使用不同的用户，那么他们将需要在JAAS配置文件中有单独的部分。  
    
**java.security.auth.login.config**
    
        用于配置Solr客户端以进行节点间通信的JAAS配置文件的路径。该参数是必需的。  
    

这是一个可以添加到bin/solr.in.sh的例子。确保将此示例更改为使用正确的主机名和keytab文件路径。  
```
SOLR_AUTH_TYPE="kerberos"
SOLR_AUTHENTICATION_OPTS="-Djava.security.auth.login.config=/home/foo/jaas-client.conf -Dsolr.kerberos.cookie.domain=192.168.0.107 -Dsolr.kerberos.cookie.portaware=true -Dsolr.kerberos.principal=HTTP/192.168.0.107@EXAMPLE.COM -Dsolr.kerberos.keytab=/keytabs/107.keytab"
```
### 具有 AES-256 加密的 KDC
如果您的KDC使用AES-256加密，则在Kerberized Solr可以与KDC交互之前，您需要将Java Cryptography Extension（JCE）无限制强制管辖权策略文件添加到您的JRE。  
  
当您在Solr日志中看到如下错误时，您将会知道这一点：“KrbException：带有HMAC SHA1-96的加密类型AES256 CTS模式不支持/启用”  
对于Java 1.8，可以在这里找到：http : //www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html。  
将 JAVA_HOME/jre/lib/security/ 与新的 local_policy. jar 一起替换为已下载包中的 local_policy，然后重新启动Solr节点。  

### 使用委托令牌

Kerberos插件可以配置为使用委托令牌，允许应用程序重复使用最终用户或其他应用程序的身份验证。  
  
对于 Solr，有几个用例可能会有帮助：  

    - 使用分布式客户端（如MapReduce），其中每个客户端可能无法访问用户的凭据。
    - 当 Kerberos 服务器上的负载很高时。委托令牌可以减少负载，因为它们在第一次请求后不访问服务器。
    - 如果需要将请求或权限委派给其他用户。

要启用委托令牌，必须定义几个参数。这些参数可以通过bin/solr启动命令在命令行中传递（请参阅“Solr控制脚本参考”以了解如何传递系统参数的详细信息），也可以在bin/solr.in.sh或者bin/solr.in.cmd中根据您的操作系统进行相应的定义。  

**solr.kerberos.delegation.token.enabled**
    
        默认情况下为<code>false</code>，设置为<code>true</code>时启用委派令牌。如果要启用令牌，则此参数是必需的。  
    
**solr.kerberos.delegation.token.kind**
    
        代表令牌的类型。默认为<code>solr-dt</code>。可能这不需要改变。目前没有其他选项可用。  
    
**solr.kerberos.delegation.token.validity**
    
        代表令牌的有效时间，以秒为单位。默认值是36000秒。  
    
**solr.kerberos.delegation.token.signer.secret.provider**
    
        代表令牌信息存储在内部。默认值是<code>zookeeper</code>，必须是授权令牌在Solr服务器上工作的位置（在SolrCloud模式下运行时）。目前没有其他选项可用。  
    
**solr.kerberos.delegation.token.signer.secret.provider.zookeper.path**
    
        秘密提供者信息所在的ZooKeeper路径。这是路径+ /安全/令牌的形式。路径可以包含chroot，如果您不使用chroot，则可以省略chroot。这个例子包括在chroot： <code>server1:9983,server2:9983,server3:9983/solr/security/token</code>。  
    
**solr.kerberos.delegation.token.secret.manager.znode.working.path**
    
        存储令牌信息的ZooKeeper路径。这是路径+ / security / zkdtsm的形式。路径可以包含chroot，如果您不使用chroot，则可以省略chroot。这个例子包括在chroot： <code>server1:9983,server2:9983,server3:9983/solr/security/zkdtsm</code>。  
    


### 启动Solr

配置完成后，可以使用bin/solr脚本启动Solr ，如下面的示例所示，仅适用于SolrCloud模式的用户。这个例子假设你修改了bin/solr.in.sh或者bin/solr.in.cmd是正确的值，但是如果你没有修改的话，你可以把系统参数和start命令一起传递。请注意，您还需要根据您的ZooKeeper节点的位置自定义该-z属性。  
```
bin/solr -c -z server1:2181,server2:2181,server3:2181/solr
```

### 测试配置

1 <li>用你的用户名做一个 kinit。例如：kinit user@EXAMPLE.COM。</li>
## 使用SolrJ与Kerberized Solr

要在SolrJ应用程序中使用Kerberos身份验证，在创建SolrClient之前，需要以下两行代码：  
```
System.setProperty("java.security.auth.login.config", "/home/foo/jaas-client.conf");
HttpClientUtil.setConfigurer(new Krb5HttpClientConfigurer());
```
您需要为客户端指定一个Kerberos服务主体，并在上面的JAAS客户端配置文件中指定相应的keytab。这个负责人应该与我们为Solr创建的服务负责人不同。  
这是一个例子：  
```
SolrJClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  keyTab="/keytabs/foo.keytab"
  storeKey=true
  useTicketCache=true
  debug=true
  principal="solrclient@EXAMPLE.COM";
};
```

### 带SolrJ的委托令牌

SolrJ也支持授权令牌，方法如下：  

    - DelegationTokenRequest 和 DelegationTokenResponse 可以用来获取、取消和续订委托令牌。
    - HttpSolrClient.Builder包括一个 withDelegationToken，用于创建使用委托令牌进行认证的HttpSolrClient 的函数。

获取授权令牌的示例代码：  
```
private String getDelegationToken(final String renewer, final String user, HttpSolrClient solrClient) throws Exception {
    DelegationTokenRequest.Get get = new DelegationTokenRequest.Get(renewer) {
      @Override
      public SolrParams getParams() {
        ModifiableSolrParams params = new ModifiableSolrParams(super.getParams());
        params.set("user", user);
        return params;
      }
    };
    DelegationTokenResponse.Get getResponse = get.process(solrClient);
    return getResponse.getDelegationToken();
  }
```
创建使用委派令牌的 HttpSolrClient：  
```
HttpSolrClient client = new HttpSolrClient.Builder("http://localhost:8983/solr").withDelegationToken(token).build();
```
创建使用委派令牌的 CloudSolrClient  
```
CloudSolrClient client = new CloudSolrClient.Builder()
                .withZkHost("http://localhost:2181")
                .withLBHttpSolrClientBuilder(new LBHttpSolrClient.Builder()
                    .withResponseParser(client.getParser())
                    .withHttpSolrClientBuilder(
                        new HttpSolrClient.Builder()
                            .withKerberosDelegationToken(token)
                    ))
                        .build();
```
Hadoop 的委托令牌响应采用 JSON 映射格式。DelegationTokenResponse 中提供了一个响应分析器。其他响应解析器可能无法很好地处理 Hadoop 响应。  
