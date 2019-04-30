## Solr使用自签名证书启启用S​​SL 
<div class="content-intro view-box ">Solr可以使用SSL对客户端和 SolrCloud 模式下的节点之间的通信进行加密。  
  
本节介绍如何使用自签名证书启用SSL。  
有关SSL证书和密钥的背景信息，请参阅http://www.tldp.org/HOWTO/SSL-Certificates-HOWTO/。  
## 基本的SSL设置

### 生成自签名证书和密钥
要生成自签名证书和将用于对服务器和客户端进行身份验证的单个密钥，我们将使用JDK keytool命令并创建一个单独的密钥库。这个密钥库也将被用作下面的信任库。为了实现这些目的，可以使用JDK附带的密钥库，并使用单独的信任库，但这些选项不在此处介绍。  
  
在二进制Solr发行版的server/etc/目录中运行以下命令。假设您的PATH中有JDK keytool实用程序，那openssl也在您的PATH中。有关Windows和Solaris的OpenSSL二进制文件，请参阅https://www.openssl.org/related/binaries.html。  
该-ext SAN=…​ keytool选项允许您指定在主机名验证期间允许的使用所有DNS名称或IP地址（但请参阅下面的内容，了解如何跳过Solr节点之间的主机名验证，以便您不必在此指定所有主机）。  
除了localhost和127.0.0.1，这个示例包括LAN IP地址192.168.1.3，用于 Solr 节点将在其上运行的计算机：  
```
keytool -genkeypair -alias solr-ssl -keyalg RSA -keysize 2048 -keypass secret -storepass secret -validity 9999 -keystore solr-ssl.keystore.jks -ext SAN=DNS:localhost,IP:192.168.1.3,IP:127.0.0.1 -dname "CN=localhost, OU=Organizational Unit, O=Organization, L=Location, ST=State, C=Country"
```
上面的命令将在当前目录中创建一个命名为solr-ssl.keystore.jks的密钥库文件。  
### 将证书和密钥转换为PEM格式以便与curl一起使用
curl不能使用JKS格式的密钥库，所以JKS密钥库需要转换为curl可以理解的PEM格式。  
  
首先使用以下keytool命令将JKS密钥库转换为PKCS12格式：  
```
keytool -importkeystore -srckeystore solr-ssl.keystore.jks -destkeystore solr-ssl.keystore.p12 -srcstoretype jks -deststoretype pkcs12
```
keytool应用程序将提示您创建目标密钥库密码和创建密钥库时设置的源密钥库密码（上面示例中的“secret”）。  
接下来，使用以下openssl命令将PKCS12格式的密钥库（包括证书和密钥）转换为PEM格式：  
```
openssl pkcs12 -in solr-ssl.keystore.p12 -out solr-ssl.pem
```
如果要在OS X Yosemite（10.10）上使用curl，则需要创建PEM格式的证书版本，如下所示：  
```
openssl pkcs12 -nokeys -in solr-ssl.keystore.p12 -out solr-ssl.cacert.pem
```
### 设置常见的SSL相关系统属性
Solr控制脚本已经设置为将与SSL相关的Java系统属性传递给JVM。要激活SSL设置，请取消注释并更新bin/solr.in.sh 中的以 SOLR_SSL_ *开始的一组属性。（或 Windows 上的bin\solr.in.cmd）。  
  
如果使用将Solr转换为Production的步骤将Solr设置为Linux上的服务，请在<code>/var/solr/solr.in.sh</code>中改为使用这些更改。  
bin / solr.in.sh示例SOLR_SSL_ *配置：  
```
SOLR_SSL_KEY_STORE=etc/solr-ssl.keystore.jks
SOLR_SSL_KEY_STORE_PASSWORD=secret
SOLR_SSL_TRUST_STORE=etc/solr-ssl.keystore.jks
SOLR_SSL_TRUST_STORE_PASSWORD=secret
# Require clients to authenticate
SOLR_SSL_NEED_CLIENT_AUTH=false
# Enable clients to authenticate (but not require)
SOLR_SSL_WANT_CLIENT_AUTH=false
# Define Key Store type if necessary
SOLR_SSL_KEY_STORE_TYPE=JKS
SOLR_SSL_TRUST_STORE_TYPE=JKS
```
当您启动Solr时，bin/solr脚本将包含这些设置，bin/solr.in.sh并将这些SSL相关的系统属性传递给JVM。  
<table><tbody><tr><td></td><td>客户端验证设置  
启用S​​OLR_SSL_NEED_CLIENT_AUTH或SOLR_SSL_WANT_CLIENT_AUTH，但不能同时使用两者。他们是相互排斥的，码头将选择其中一个可能不是你所期望的。</td></tr></tbody></table>同样，当您在Windows上启动Solr时，bin\solr.in.cmd中的bin\solr.cmd脚本将包含以下设置：取消注释并更新SOLR_SSL_*以将这些SSL相关的系统属性传递给JVM 开始的属性集：  
bin \ solr.in.cmd示例SOLR_SSL_ *配置：  
```
set SOLR_SSL_KEY_STORE=etc/solr-ssl.keystore.jks
set SOLR_SSL_KEY_STORE_PASSWORD=secret
set SOLR_SSL_TRUST_STORE=etc/solr-ssl.keystore.jks
set SOLR_SSL_TRUST_STORE_PASSWORD=secret
REM Require clients to authenticate
set SOLR_SSL_NEED_CLIENT_AUTH=false
REM Enable clients to authenticate (but not require)
set SOLR_SSL_WANT_CLIENT_AUTH=false
```
### 使用SSL运行单节点Solr
使用下面的命令启动Solr; 默认情况下，客户端不需要认证：  
- * nix 命令：
- 
```
bin/solr -p 8984
```
- Windows命令：
- 
```
bin\solr.cmd -p 8984
```

## SSL与SolrCloud
本节介绍如何运行没有初始集合的双节点SolrCloud集群以及单节点外部ZooKeeper。以下命令假定您已经创建了上述密钥库。  
### 配置ZooKeeper
  
管理员不支持像 Solr 这样的客户进行加密通信。有几个相关的 JIRA 门票, SSL 支持正在计划/工作: ZOOKEEPER-235;ZOOKEEPER-236;ZOOKEEPER-1000;和 ZOOKEEPER-2120。  
ZooKeeper不支持像Solr这样的客户端的加密通信。有几个相关的JIRA标签正在计划/处理SSL支持：ZOOKEEPER-235 ; ZOOKEEPER-236 ; ZOOKEEPER-1000 ; 和ZOOKEEPER-2120。  
在启动任何SolrCloud节点之前，您必须在ZooKeeper中配置solr集群属性，以便Solr节点知道通过SSL进行通信。  
  
本节假设您已经在本地主机上的端口2181上创建并启动了单节点外部ZooKeeper - 请参阅设置外部ZooKeeper集成。  
在任何Solr节点启动之前，都需要将urlScheme集群属性设置为https。下面的示例使用二进制Solr发行版附带的zkcli工具来执行此操作：  
- * nix 命令：
- 
```
server/scripts/cloud-scripts/zkcli.sh -zkhost localhost:2181 -cmd clusterprop -name urlScheme -val https
```
- Windows 命令：
- 
```
server\scripts\cloud-scripts\zkcli.bat -zkhost localhost:2181 -cmd clusterprop -name urlScheme -val https
```
如果您已经设置了您的ZooKeeper的集群使用了Solr的chroot环境，确保您使用zkcli的正确的zkhost字符串，如：-zkhost localhost:2181/solr。  
### 使用SSL运行SolrCloud

#### 创建两个节点的Solr主目录
为两个SolrCloud节点中的每一个节点创建两个server/solr/副本，作为Solr主目录：  
- * nix 命令：
- 
```
mkdir cloud
cp -r server/solr cloud/node1
cp -r server/solr cloud/node2
```
- Windows 命令：
- 
```
mkdir cloud
xcopy /E server\solr cloud\node1\
xcopy /E server\solr cloud\node2\
```

#### 启动第一个Solr节点
接下来，在端口8984上启动第一个Solr节点。如果在通过本页上一节的工作时启动它，请务必先停止独立服务器。  
- * nix 命令：
- 
```
bin/solr -cloud -s cloud/node1 -z localhost:2181 -p 8984
```
- Windows 命令：
- 
```
bin\solr.cmd -cloud -s cloud\node1 -z localhost:2181 -p 8984
```
请注意, 使用-s 选项可以设置 node1 的 Solr 主目录的位置。  
如果您创建的 SSL 密钥没有所有 solr 节点将运行的 DNS 名称/IP 地址, 则可以告诉 solr 通过将 solr.ssl.checkPeerName 系统属性设置为 false 来跳过 solr 节点间通信的主机名验证:  
  
  
  
请注意，使用该-s选项可以设置 node1 的 Solr 主目录的位置。  
如果您创建了没有运行Solr节点的所有DNS names/IP 地址的 SSL 密钥，则可以告诉 solr 通过将solr.ssl.checkPeerName系统属性设置为false来跳过Solr-node通信跳过主机名验证。  
* nix 命令：  
```
bin/solr -cloud -s cloud/node1 -z localhost:2181 -p 8984 -Dsolr.ssl.checkPeerName=false
```
Windows 命令：  
```
bin\solr.cmd -cloud -s cloud\node1 -z localhost:2181 -p 8984 -Dsolr.ssl.checkPeerName=false
```
#### 启动第二个Solr节点
最后，在端口7574上启动第二个Solr节点 - 再次跳过主机名验证，添加-Dsolr.ssl.checkPeerName=false;  
- * nix 命令：
- 
```
bin/solr -cloud -s cloud/node2 -z localhost:2181 -p 7574
```
- Windows 命令：
- 
```
bin\solr.cmd -cloud -s cloud\node2 -z localhost:2181 -p 7574
```

## 客户端操作示例
  
在 OS X 小牛 (10.9) 上卷曲已降级的 SSL 支持。有关允许 one-way SSL 的更多信息和变通方法, 请参见 http://curl.haxx.se/mail/archive-2013-10/0036.html。在 OS X 优胜美地 (10.10) 被改进-2 方式 SSL 是可能的-看见 http://curl.haxx.se/mail/archive-2014-10/0053.html。  
  
以下各节中的卷曲命令在 OS X 优胜美地 (10.10) 上的系统卷曲不会工作。相反, 使用-e 参数提供的证书必须是 PKCS12 格式的, 并且提供了-cacert 参数的文件必须只包含 CA 证书, 并且没有密钥 (请参见上面有关创建此文件的说明):  
  
  
在OS X Mavericks（10.9）上 curl 已经降级了SSL支持。有关允许单向SSL的更多信息和解决方法，请参阅http://curl.haxx.se/mail/archive-2013-10/0036.html。在OS X Yosemite（10.10）上 curl 得到改进 - 可以使用双向SSL - 请参阅http://curl.haxx.se/mail/archive-2014-10/0053.html。  
以下各节中的 curl 命令在OS X Yosemite（10.10）中的系统无法使用以下部分中的curl命令。相反。使用 -E 参数提供的证书必须是 PKCS12 格式的。并且提供了-cacert 参数的文件必须只包含 CA 证书，并且不包含任何密钥（有关创建此文件的说明，请参阅上文）：  
```
curl -E solr-ssl.keystore.p12:secret --cacert solr-ssl.cacert.pem ...
```
如果您的操作系统不包含 curl，您可以在这里下载二进制文件：http ://curl.haxx.se/download.html  
### 使用bin / solr创建一个SolrCloud集合
使用默认的configset（_default）创建名为mycollection的 2-shard，replicationFactor = 1集合：  
  
<p/>- * nix 命令：
- 
```
bin/solr create -c mycollection -shards 2
```
<p/>- Windows 命令：
- 
```
bin\solr.cmd create -c mycollection -shards 2
```
该create操作将把您的包含文件中设置的 SOLR_SSL_ * 属性传递给用于创建集合的 SolrJ 代码。  
### 使用curl检索SolrCloud群集状态
要获得最终的群集状态（再次，如果您尚未启用客户端身份验证，请删除该-E solr-ssl.pem:secret选项）：  
```
curl -E solr-ssl.pem:secret --cacert solr-ssl.pem "https://localhost:8984/solr/admin/collections?action=CLUSTERSTATUS&amp;indent=on"
```
您应该得到这样的回应：  
```
{
  "responseHeader":{
    "status":0,
    "QTime":2041},
  "cluster":{
    "collections":{
      "mycollection":{
        "shards":{
          "shard1":{
            "range":"80000000-ffffffff",
            "state":"active",
            "replicas":{"core_node1":{
                "state":"active",
                "base_url":"https://127.0.0.1:8984/solr",
                "core":"mycollection_shard1_replica1",
                "node_name":"127.0.0.1:8984_solr",
                "leader":"true"}}},
          "shard2":{
            "range":"0-7fffffff",
            "state":"active",
            "replicas":{"core_node2":{
                "state":"active",
                "base_url":"https://127.0.0.1:7574/solr",
                "core":"mycollection_shard2_replica1",
                "node_name":"127.0.0.1:7574_solr",
                "leader":"true"}}}},
        "maxShardsPerNode":"1",
        "router":{"name":"compositeId"},
        "replicationFactor":"1"}},
    "properties":{"urlScheme":"https"}}}
```
### 索引文件使用post.jar
使用post.jar索引一些示例文档到上面创建的SolrCloud集合：  
```
cd example/exampledocs
java -Djavax.net.ssl.keyStorePassword=secret -Djavax.net.ssl.keyStore=../../server/etc/solr-ssl.keystore.jks -Djavax.net.ssl.trustStore=../../server/etc/solr-ssl.keystore.jks -Djavax.net.ssl.trustStorePassword=secret -Durl=https://localhost:8984/solr/mycollection/update -jar post.jar *.xml
```
### 使用curl查询
使用curl来查询上面创建的SolrCloud集合，从包含上面创建的PEM格式化证书和密钥的目录（例如：example/etc/） - 如果您还没有启用客户端身份验证（系统属性-Djetty.ssl.clientAuth=true)，那么您可以删除-E solr-ssl.pem:secret选项：  
  
```
curl -E solr-ssl.pem:secret --cacert solr-ssl.pem "https://localhost:8984/solr/mycollection/select?q=*:*"
```
### 使用CloudSolrClient索引文档
从一个使用SolrJ的java客户端索引一个文档。在下面的代码中，javax.net.ssl.*系统属性是以编程方式设置的，但是您可以在java命令行中指定它们，如上post.jar例所示：  
```
System.setProperty("javax.net.ssl.keyStore", "/path/to/solr-ssl.keystore.jks");
System.setProperty("javax.net.ssl.keyStorePassword", "secret");
System.setProperty("javax.net.ssl.trustStore", "/path/to/solr-ssl.keystore.jks");
System.setProperty("javax.net.ssl.trustStorePassword", "secret");
String zkHost = "127.0.0.1:2181";
CloudSolrClient client = new CloudSolrClient.Builder().withZkHost(zkHost).build();
client.setDefaultCollection("mycollection");
SolrInputDocument doc = new SolrInputDocument();
doc.addField("id", "1234");
doc.addField("name", "A lovely summer holiday");
client.add(doc);
client.commit();
```
