## ZooKeeper访问控制 
<div class="content-intro view-box ">本节介绍如何在Solr中使用ZooKeeper访问控制列表（ACL）。有关ZooKeeper ACL的信息，请参阅：http://zookeeper.apache.org/doc/r3.4.10/zookeeperProgrammers.html#sc_ZooKeeperAccessControl 的ZooKeeper文档。
      
  

## 关于ZooKeeper ACL

SolrCloud使用ZooKeeper来共享信息和进行协调。
      
  
本节介绍如何配置Solr以将更多限制性的ACL添加到其创建的ZooKeeper内容，以及如何告知Solr有关访问ZooKeeper中内容所需的凭据。如果你想在你的ZooKeeper节点中使用ACL，你必须激活这个功能。默认情况下，Solr行为是开放不安全的ACL，并且不使用任何凭据。  
存储在ZooKeeper中的内容对于SolrCloud集群的运行至关重要。开放访问ZooKeeper上的SolrCloud内容可能会导致各种问题。例如：  

    - 更改配置可能会导致Solr失败或以非预期的方式运行。
    - 将群集状态信息更改为错误或不一致的情况可能会使SolrCloud群集行为异常。
    - 添加由监督员执行的删除集合作业将导致数据从群集中删除。

如果您将ZooKeeper集合的访问权限授予您不信任的实体，或者您希望减少由于下列原因导致的错误操作的风险，则可能需要使用Solr启用ZooKeeper ACL：  

    - 进入你系统的恶意软件。
    - 其他使用相同ZooKeeper集合的系统（一个“bad thing”可能是偶然发生的）。

如果你认为ZooKeeper中有些东西不是每个人都应该知道的，你甚至可能想要限制读取权限。或者你可能只是在需要知道的基础上进行一般性的工作。  
保护ZooKeeper本身可能意味着许多不同的事情。本节是关于保护ZooKeeper中的Solr内容。ZooKeeper内容基本上一直存在于磁盘上，部分地存在于ZooKeeper进程的内存中。本节不涉及在存储或ZooKeeper进程级别保护ZooKeeper数据 - 这是ZooKeeper需要处理的。  
但是这个内容也可以通过ZooKeeper API“外部”提供。外部进程可以连接到ZooKeeper并创建/更新/删除/读取内容；例如，SolrCloud集群中的Solr节点想要创建/更新/删除/读取，并且SolrJ客户端想要从集群中读取。创建/更新内容来设置内容的ACL是外部进程的责任。ACL描述谁可以读取，更新，删除，创建等。ZooKeeper中的每个信息（znode / content）都有自己的一组ACL，并且不可能继承或共享。Solr中的默认行为是在其创建的所有内容上添加一个ACL -
    一个允许任何人执行任何操作的权限（在ZooKeeper术语中称为“开放式不安全ACL（open-unsafe ACL）”）。  

## 如何启用ACL

我们希望能够：
      
  
1 <li>控制Solr用于其ZooKeeper连接的凭据。凭证用于获取在ZooKeeper中执行操作的权限。</li>2 <li>从“外部”控制它，这样您就不必修改或重新编译Solr代码即可将其打开。</li>
Solr节点，客户端和工具（例如ZkCLI）总是使用一个称为SolrZkClient的java类来处理他们的ZooKeeper的东西。这里所描述的解决方案的实现就是要改变的SolrZkClient。如果您在您的应用程序中使用SolrZkClient，则下面的描述也适用于您的应用程序。  

### 控制凭证

通过在solr.xml的&lt;solrcloud&gt;部分中配置zkCredentialsProvider属性到类的名称（在类别路径上）来实现 zkCredentialsProvider 接口，这可以控制将使用哪个凭据提供程序。在Solr分布中的server/solr/solr.xml，定义了zkCredentialsProvider如果定义了它，它将采用同名zkCredentialsProvider系统属性的值（例如，通过取消注释在solr.in.sh/.cmd中的SOLR_ZK_CREDS_AND_ACLS环境变量定义-
    参见下文），或者如果不定义，则默认为DefaultZkCredentialsProvider实现。
      
  

#### 开箱凭证实现

你总是可以自己实现，但Solr有两个实现：
      
  

    - org.apache.solr.common.cloud.DefaultZkCredentialsProvider：它的getCredentials()返回一个长度为零的列表，或者“没有使用凭据”。这是默认的。
    - org.apache.solr.common.cloud.VMParamsSingleSetCredentialsDigestZkCredentialsProvider：这使您可以使用系统属性来定义凭据。它最多支持一组证书。<ul><li>架构是“digest”。用户名和密码由系统属性zkDigestUsername和zkDigestPassword定义。如果提供了用户名和密码，则这组凭证将被添加到getCredentials()返回的凭证列表中。
- 如果上述一组凭证未添加到列表中，则此实现将回退到默认行为并使用DefaultZkCredentialsProvider（空）凭证列表。
</li>
</ul>
### 控制ACL

通过在solr.xml的&lt;solrcloud&gt;部分中配置zkACLProvider属性到类的名称（在类别路径上）来实现 ZkACLProvider 接口，将会控制要添加的ACL。在Solr分布中的server/solr/solr.xml，定义了zkACLProvider，如果定义了它，它将采用同名zkACLProvider系统属性的值（例如，通过取消注释在solr.in.sh/.cmd中的SOLR_ZK_CREDS_AND_ACLS环境变量定义 - 参见下文），或者如果不定义，则默认DefaultZkACLProvider执行。
      
  

#### 开箱即用的ACL实现

你总是可以让你自己实现，但Solr有两个实现：  

    - org.apache.solr.common.cloud.DefaultZkACLProvider：它为所有zNodePath-s 返回一个长度为1的列表。列表中的单个ACL条目是“不安全的”。这是默认的。
    - org.apache.solr.common.cloud.VMParamsAllAndReadonlyDigestZkACLProvider：这使您可以使用系统属性来定义ACL。它的 getACLsToAdd () 实现对任何东西都不使用 zNodePath，所以所有的znode都会得到相同的一组ACL。它支持添加以下一个或两个选项：<ul><li>允许执行所有操作的用户。<ul style="list-style-type:circle"><li>该许可是“ALL”（对应于所有的CREATE，READ，WRITE，DELETE，和ADMIN），而架构是“digest”。
- 用户名和密码分别由系统属性zkDigestUsername和zkDigestPassword定义。
- 除非提供用户名和密码，否则此ACL不会添加到ACL列表中。
</li></ul>- 只允许执行读取操作的用户。<ul style="list-style-type:circle"><li>权限是READ以及架构是digest。
- 用户名和密码分别由系统属性zkDigestReadonlyUsername和zkDigestReadonlyPassword定义。
- 除非提供用户名和密码，否则此ACL不会添加到ACL列表中。
</li></ul></li>
    <li>org.apache.solr.common.cloud.SaslZkACLProvider：需要SASL身份验证。在使用SASL时，为系统属性solr.authorization.superuser（默认: solr）中指定的用户授予所有权限，并为其他任何人提供读取权限。设计用于配置已经设置且不会被修改的设置，或者通过Solr API控制配置更改的位置。这个提供程序将在kerberos环境中用于管理。在这样的环境中，管理员希望Solr使用SASL对ZooKeeper进行身份验证，因为这只能通过Kerberos与ZooKeeper进行身份验证。</li>
</ul>
如果没有上述ACL添加到列表中，则默认情况下将使用 DefaultZkACLProvider 的（空）ACL列表。
      
  
注意系统属性名称与证书提供程序VMParamsSingleSetCredentialsDigestZkCredentialsProvider（如上所述）重叠。这是为了让两个供应商以一种很好的方式进行协作：通过限制两个用户（一个管理员用户和一个只读用户），我们始终保护对内容的访问权限，而且我们始终使用与相同管理员用户相对应的凭据进行连接，基本上这样我们可以对我们自己创建的内容/ znode做任何事情。  
您可以将只读凭据提供给SolrCloud群集的“客户端” - 例如，供SolrJ客户端使用。他们将能够读取运行SolrJ客户端所需的任何内容，但是他们将无法修改ZooKeeper中的任何内容。  

### Solr脚本中的ZooKeeper ACL

有两个影响ZooKeeper ACL的脚本：  

    - 对于* nix系统：bin/solr＆server/scripts/cloud-scripts/zkcli.sh
    - 对于Windows系统：bin/solr.cmd＆server/scripts/cloud-scripts/zkcli.bat

这些Solr脚本可以通过设置相应的系统属性来启用ZK ACL：取消注释以下内容，并将密码替换为您选择的密码，以便在以下文件中启用上述VM参数ACL和凭证提供程序：  
<b>solr.in.sh：</b>
  
```
# Settings for ZK ACL
#SOLR_ZK_CREDS_AND_ACLS="-DzkACLProvider=org.apache.solr.common.cloud.VMParamsAllAndReadonlyDigestZkACLProvider \
#  -DzkCredentialsProvider=org.apache.solr.common.cloud.VMParamsSingleSetCredentialsDigestZkCredentialsProvider \
#  -DzkDigestUsername=admin-user -DzkDigestPassword=CHANGEME-ADMIN-PASSWORD \
#  -DzkDigestReadonlyUsername=readonly-user -DzkDigestReadonlyPassword=CHANGEME-READONLY-PASSWORD"
#SOLR_OPTS="$SOLR_OPTS $SOLR_ZK_CREDS_AND_ACLS"
```
<b>solr.in.cmd：</b>
  
```
REM Settings for ZK ACL
REM set SOLR_ZK_CREDS_AND_ACLS=-DzkACLProvider=org.apache.solr.common.cloud.VMParamsAllAndReadonlyDigestZkACLProvider ^
REM  -DzkCredentialsProvider=org.apache.solr.common.cloud.VMParamsSingleSetCredentialsDigestZkCredentialsProvider ^
REM  -DzkDigestUsername=admin-user -DzkDigestPassword=CHANGEME-ADMIN-PASSWORD ^
REM  -DzkDigestReadonlyUsername=readonly-user -DzkDigestReadonlyPassword=CHANGEME-READONLY-PASSWORD
REM set SOLR_OPTS=%SOLR_OPTS% %SOLR_ZK_CREDS_AND_ACLS%
```
<b>zkcli.sh：</b>
  
```
# Settings for ZK ACL
#SOLR_ZK_CREDS_AND_ACLS="-DzkACLProvider=org.apache.solr.common.cloud.VMParamsAllAndReadonlyDigestZkACLProvider \
#  -DzkCredentialsProvider=org.apache.solr.common.cloud.VMParamsSingleSetCredentialsDigestZkCredentialsProvider \
#  -DzkDigestUsername=admin-user -DzkDigestPassword=CHANGEME-ADMIN-PASSWORD \
#  -DzkDigestReadonlyUsername=readonly-user -DzkDigestReadonlyPassword=CHANGEME-READONLY-PASSWORD"
```
<b>zkcli.bat：</b>
  
```
REM Settings for ZK ACL
REM set SOLR_ZK_CREDS_AND_ACLS=-DzkACLProvider=org.apache.solr.common.cloud.VMParamsAllAndReadonlyDigestZkACLProvider ^
REM  -DzkCredentialsProvider=org.apache.solr.common.cloud.VMParamsSingleSetCredentialsDigestZkCredentialsProvider ^
REM  -DzkDigestUsername=admin-user -DzkDigestPassword=CHANGEME-ADMIN-PASSWORD ^
REM  -DzkDigestReadonlyUsername=readonly-user -DzkDigestReadonlyPassword=CHANGEME-READONLY-PASSWORD
```

## 更改ACL方案

在运行Solr集群的整个过程中，您可能决定从一个不安全的ZooKeeper移动到一个安全的实例。更改在在solr.xml中配置的zkACLProvider将确保新创建的节点是安全的，但不会保护已有的数据。要修改所有现有的ACL，可以在Solr的ZkCLI中使用该updateacls命令。首先对定义在server/scripts/cloud-scripts/zkcli.sh（或在Windows上的zkcli.bat）上的SOLR_ZK_CREDS_AND_ACLS环境变量取消注释，然后填写admin-user和readonly-user的密码
    - 见上面 - 然后运行server/scripts/cloud-scripts/zkcli.sh -cmd updateacls /zk-path，或者在Windows上运行：server\scripts\cloud-scripts\zkcli.bat cmd updateacls /zk-path。
      
  
在ZK中更改ACL时，只能在SolrCloud群集停止时执行。尝试在Solr运行时执行此操作可能会导致状态不一致，并且某些节点无法访问。
      
  
VM属性：zkACLProvider和zkCredentialsProvider（包含在zkcli.sh/.bat的SOLR_ZK_CREDS_AND_ACLS环境变量中）控制着转换：  

    - 凭证提供程序必须是在节点上具有当前管理权限的凭证提供程序。省略时，该进程将不使用任何凭据（适用于不安全的配置）。
    - ACL提供程序将用于计算新的ACL。省略时，该过程将为所有用户设置所有权限，从而消除存在的任何安全性。

zkcli.sh/.bat中未注释的SOLR_ZK_CREDS_AND_ACLS环境变量将凭据和ACL提供程序设置为VMParamsSingleSetCredentialsDigestZkCredentialsProvider和VMParamsAllAndReadonlyDigestZkACLProvider实现。  
