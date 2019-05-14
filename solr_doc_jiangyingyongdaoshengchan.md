# 将Solr应用到生产

本节提供有关如何设置 Solr 以在 *nix 平台（如 Ubuntu）的生产中运行的指南。具体来说，我们将介绍在 Linux 主机上运行单个 Solr 实例的过程，然后提供有关如何支持在同一主机上运行的多个 Solr 节点的提示。  


## 服务安装脚本

Solr 包含一个服务安装脚本（bin/install_solr_service.sh），它可以帮助您在 Linux 上将 Solr 作为服务安装。目前，该脚本仅支持 CentOS、Debian、Red Hat、SUSE 和 Ubuntu Linux 发行版。在运行脚本之前，您需要确定一些关于您的设置的参数。具体而言，您需要决定 Solr 的安装位置以及哪些系统用户应该是 Solr 文件和进程的所有者。  

### 规划目录结构

我们建议将实时Solr文件（例如日志和索引文件）与Solr分发包中包含的文件分开，因为这样可以更容易地升级Solr，并且被认为是作为系统管理员遵循的良好做法。  

#### Solr 安装目录

默认情况下，服务安装脚本将解压缩到：/opt。当您运行安装脚本时，可以使用 -i 选项更改此位置。该脚本还会创建一个指向 Solr 的版本化目录的符号链接。例如，如果您为 Solr 7.0.0 运行安装脚本，则将使用以下目录结构：

```sh
/opt/solr-7.0.0
/opt/solr > /opt/solr-7.0.0
```

使用符号链接可以使任何脚本不依赖于特定的 Solr 版本。如果您需要升级到更高版本的 Solr，则可以更新符号链接以指向升级版本的 Solr。我们将使用 /opt/solr 在本页的其余部分中引用 Solr 安装目录。  

#### 可写文件的独立目录

您还应该将可写的 Solr 文件分隔到不同的目录中；默认情况下，安装脚本使用：/var/solr，但您可以使用 -d 选项覆盖此位置。采用这种方法，在 /opt/solr 中的文件将保持不变，并且所有在 Solr 运行时更改的文件都将处于 /var/solr 下面。  

### 创建 Solr 用户

出于安全考虑，建议不要以 root 身份运行 Solr，并且 "控制脚本启动" 命令将拒绝这样做。因此，您应该确定将拥有所有 Solr 文件和正在运行的 Solr 进程的系统用户的用户名。默认情况下，安装脚本将创建 solr 用户，但您可以使用 -u 选项覆盖此设置。如果您的组织对创建新的用户帐户有特定的要求，那么您应该在运行脚本之前创建用户。安装脚本将使 Solr 用户成为 /opt/solr和/var/solr 目录的所有者。  
现在，您可以运行安装脚本了。  


### 运行 Solr 安装脚本

要运行该脚本，您需要下载最新的 Solr 分发包，然后执行以下操作：  

```sh
tar xzf solr-7.0.0.tgz solr-7.0.0/bin/install_solr_service.sh --strip-components=2
```

前面的命令 install_solr_service.sh 将档案中的脚本提取到当前目录中。如果在 Red Hat 上安装，请确保在运行 Solr 安装脚本之前安装了 lsof（sudo yum install lsof）。安装脚本必须以 root 身份运行：

```sh
sudo bash ./install_solr_service.sh solr-7.0.0.tgz
```

默认情况下，该脚本将分发归档文件提取到 /opt，配置 Solr 将文件写入 /var/solr，并以 solr 用户身份运行 Solr 。因此，下面的命令产生与上一个命令相同的结果：

```
sudo bash ./install_solr_service.sh solr-7.0.0.tgz -i /opt -d /var/solr -u solr -s solr -p 8983
```

您可以使用传递给安装脚本的选项自定义服务名称、安装目录、端口和所有者。要查看可用选项，请执行以下操作：

```
sudo bash ./install_solr_service.sh -help
```

脚本完成后，Solr 将作为服务安装并在服务器的后台运行（在端口 8983 上）。为了验证，您可以这样做：

```
sudo service solr status
```

如果您不想立即启动服务，请传递 -n 选项。然后，您可以稍后手动启动服务，例如在完成配置设置之后。  
我们将介绍一些您可以进行的其他配置设置，以便稍后微调您的 Solr 设置。在继续之前，让我们仔细看一下安装脚本执行的步骤。这样可以帮助您更好地了解并阅读本指南中的其他页面，帮助您了解有关 Solr 安装的重要细节：例如当一个页面提到 Solr 主页时，您就会知道系统中的具体位置。  

#### Solr 主目录

Solr 主目录（不要与 Solr 安装目录混淆）是 Solr 使用索引文件管理核心目录的地方。默认情况下，安装脚本使用：/var/solr/data。如果在安装脚本中使用 -d 选项，则这将更改为给定 -d 选项的位置中的 data 子目录。请花些时间检查系统上的 Solr 主目录的内容。如果您没有在 ZooKeeper 中存储 solr.xml，则主目录必须包含一个 solr.xml 文件。当 Solr 启动时，Solr 控制脚本将使用-Dsolr.solr.home=…​系统属性传递主目录的位置。  

#### 环境重写包含文件

服务安装脚本将创建一个特定于环境的包含文件，该文件将重写 bin/solr 脚本所使用的默认值。使用包含文件的主要优点是它提供了一个单一的位置，在这个位置上定义了所有特定于环境的重写。请花点时间检查 /etc/default/solr.in.sh 文件的内容，这是安装脚本设置的默认路径。如果您在安装脚本中使用 -s 选项更改服务的名称，则文件名的第一部分将会不同。对于名为 solr-demo 的服务，该文件将被命名为 /etc/default/solr-demo.in.sh。有很多设置可以用这个文件重写。但是，这个脚本至少需要定义 SOLR_PID_DIR 和 SOLR_HOME 变量，比如：  

```
SOLR_PID_DIR=/var/solr
SOLR_HOME=/var/solr/data
```

该 SOLR_PID_DIR 变量设置控制脚本将写入包含 Solr 服务器进程 ID 的文件的目录。  

#### 日志设置

Solr 使用 Apache Log4J 进行日志记录。安装脚本复制 /opt/solr/server/resources/log4j.properties 到 /var/solr/log4j.properties。请花一点时间通过在 /etc/default/solr.in.sh 中检查以下设置以验证 Solr 包含文件是否配置为将日志发送到正确的位置：

```
LOG4J_PROPS=/var/solr/log4j.properties
SOLR_LOGS_DIR=/var/solr/logs
```

有关 Log4J 配置的详细信息，请参阅: 配置日志记录。


#### init.d 脚本

在 Linux 上运行 Solr 等服务时，通常需要设置 init.d 脚本，以便系统管理员可以使用服务工具来控制 Solr，例如：service solr start。安装脚本创建一个非常基本的init.d 脚本来帮助您入门。请花点时间检查 /etc/init.d/solr 文件，这是安装脚本设置的默认脚本名称。如果在安装脚本中使用 -s 选项更改服务的名称，则文件名将会不同。请注意，根据传递给安装脚本的参数为您的环境设置了以下变量：
  
```
SOLR_INSTALL_DIR=/opt/solr
SOLR_ENV=/etc/default/solr.in.sh
RUNAS=solr
```

SOLR_INSTALL_DIR 和 SOLR_ENV 变量应该是不言而喻的。该 RUNAS 变量设置 Solr 进程的所有者，例如 solr；如果不设置此值，脚本将以 root 身份运行 Solr ，这是不建议用于生产的。您可以以 root 身份使用 /etc/init.d/solr 脚本来启动 Solr，执行以下操作：

```
service solr start
```

该 /etc/init.d/solr 脚本还支持停止、重新启动和状态命令。请记住，Solr 附带的初始化脚本非常基本，旨在向您展示如何将 Solr 设置为服务。但是，使用更高级的工具（如 supervisord 或 upstart）来控制 Solr 作为 Linux 上的服务也很常见。在展示如何将 Solr 与 supervisord 等工具整合在一起超出本指南的范围时，init.d/solr脚本应该提供足够的指导来帮助您入门。而且，安装脚本将 Solr 服务设置为在主机初始化时自动启动。  

### 进度检查

在下一节中，我们将介绍一些其他的环境设置，以帮助您微调您的生产设置。但是，在我们继续之前，让我们回顾一下迄今为止取得的成就。具体来说，你应该能够使用 /etc/init.d/solr 控制 Solr。请验证以下命令是否与您的安装程序一起使用：

```
sudo service solr restart
sudo service solr status
```

status 命令应该提供一些关于正在运行的 Solr 节点的基本信息，如：

```
Solr process PID running on port 8983
{
  "version":"5.0.0 - ubuntu - 2014-12-17 19:36:58",
  "startTime":"2014-12-19T19:25:46.853Z",
  "uptime":"0 days, 0 hours, 0 minutes, 8 seconds",
  "memory":"85.4 MB (%17.4) of 490.7 MB"
}
```

如果该 status 命令不成功，请在 /var/solr/logs/solr.log 中查找错误消息。  

## 微调您的生产设置

### 内存和 GC 设置

默认情况下，bin/solr 脚本将最大 Java 堆大小设置为 512M（-Xmx512m），这对于 Solr 入门是很好的。对于生产，您将希望根据您的搜索应用程序的内存需求增加最大堆大小；对于生产服务器，10 到 20 千兆字节的值并不少见。当您需要更改 Solr 服务器的内存设置时，请使用 SOLR_JAVA_MEM 包含文件中的变量，例如：

```sh
SOLR_JAVA_MEM="-Xms10g -Xmx10g"
```

此外，Solr 控制脚本还附带一组预先配置的 Java 垃圾收集设置，这些设置对于许多不同的工作负载都显示出与 Solr 的良好配合。但是，这些设置可能不适用于您对Solr 的具体使用。因此，您可能需要更改 GC 设置，这也应该使用 /etc/default/solr.in.sh 包含文件中的 GC_TUNE 变量来完成。有关调整内存和垃圾收集设置的更多信息，请参阅：JVM 设置。  

#### 内存不足关闭的钩子(shutdown hook)

bin/solr 脚本注册的 bin/oom_solr.sh 脚本将被 JVM 调用，如果出现一个 OutOfMemoryError。该 oom_solr.sh 脚本将发出kill -9给发生OutOfMemoryError的solr进程。在 SolrCloud 模式下运行时建议使用此行为，以便立即通知 ZooKeeper 某个节点遇到不可恢复的错误。请花点时间检查 /opt/solr/bin/oom_solr.sh 脚本的内容，以便熟悉脚本如果由 JVM 调用时将执行的操作。  

### 使用 SolrCloud 进行生产

要以 SolrCloud 模式运行 Solr，您需要在包含文件中设置 ZK_HOST 变量，以指向您的 ZooKeeper 集合。在生产环境中不支持运行嵌入式 ZooKeeper。例如，如果您在默认客户端端口 2181（zk1，zk2 和 zk3）上的以下三台主机上托管 ZooKeeper 集成，则可以设置：

```sh
ZK_HOST=zk1,zk2,zk3
```

当 ZK_HOST 变量被设置时，Solr 将以“cloud”模式启动。  

#### ZooKeeper chroot

如果您使用的是其他系统共享的 ZooKeeper 实例，建议使用 ZooKeeper 的 chroot 支持来隔离 SolrCloud znode 树。例如，要确保 SolrCloud 创建的所有 znode都存储在 /solr 下面，您可以在 ZK_HOST 连接字符串的末尾放置 /solr，例如：

```sh
ZK_HOST=zk1,zk2,zk3/solr
```

首次使用 chroot 之前，您需要使用 Solr 控制脚本在 ZooKeeper 中创建根路径（znode）。我们可以使用 mkroot命令：

```
bin/solr zk mkroot /solr -z &lt;ZK_node&gt;:&lt;ZK_PORT&gt;
```

>Tip：你还想用现有的 solr_home 引导 ZooKeeper，你可以改为使用 zkcli.sh 或zkcli.bat bootstrap命令，如果它不存在，也会创建 chroot 路径。有关更多信息，请参阅命令行实用程序。


### Solr 主机名

使用 SOLR_HOST 包含文件中的变量来设置 Solr 服务器的主机名。

```
SOLR_HOST=solr1.example.com
```

建议设置 Solr 服务器的主机名，尤其是在 SolrCloud 模式下运行时，因为这决定了在向 ZooKeeper 注册时节点的地址。  

### 重写solrconfig.xml中的设置

Solr 允许使用 -Dproperty=value 语法在启动时传递的 Java 系统属性重写配置属性。例如，在 solrconfig.xml 中，默认的 "自动软提交" 设置被设置为：

```xml
<autoSoftCommit>
  <maxTime>${solr.autoSoftCommit.maxTime:-1}</maxTime>
<autoSoftCommit>
```

一般来说，无论何时在使用 ${solr.PROPERTY:DEFAULT_VALUE} 语法的 Solr 配置文件中看到一个属性，都可以使用 Java 系统属性重写它。例如，要将 soft-commits 的 maxTime 设置为10秒，则可以使用以下命令启动 Solr -Dsolr.autoSoftCommit.maxTime=10000，例如：

```sh
bin/solr start -Dsolr.autoSoftCommit.maxTime=10000
```

该 bin/solr 脚本只是在启动时将以 -D 开头的选项传递给 JVM。为了在生产环境中运行，我们建议在 include 文件中定义 SOLR_OPTS 的变量中设置这些属性。按照我们的 soft-commit 例子，在 /etc/default/solr.in.sh 中，你可以这样做：

```sh
SOLR_OPTS="$SOLR_OPTS -Dsolr.autoSoftCommit.maxTime=10000"
```

## 每个主机运行多个Solr节点

该 bin/solr 脚本能够在一台机器上运行多个实例，但对于典型的安装，这不是推荐的设置。每个额外的实例都需要额外的 CPU 和内存资源。单个实例可以很容易地处理多个索引。

>Tip：何时忽略该建议？
对于每个建议，都有例外。对于上面的建议，该异常在讨论极限可伸缩性时主要适用。在一台主机上运行多个 Solr 节点的最佳原因是减少了对非常大的堆的需求。
当 Java 堆变得非常大时，即使启动脚本默认提供了 GC 调整，也可能导致非常长的垃圾回收暂停。堆被认为是“非常大”的确切点将取决于如何使用 Solr。这意味着没有可作为阈值的硬数字，但是如果你的堆达到了16到32千兆字节的邻域，那么可能是考虑拆分节点的时候了。理想情况下，这将意味着更多的机器，但预算的限制可能会使这一切不可能。
一旦堆达到32GB，还有另外一个问题。在32GB以下，Java 能够使用压缩的指针，但是在那之上，需要更大的指针，它使用更多的内存并且减慢了 JVM 的速度。
由于潜在的垃圾收集问题以及在32GB时发生的特定问题，如果单个实例需要64GB的堆，那么如果机器设置了两个节点，每个节点的堆大小为31GB，则性能可能会大大提高。

如果您的用例需要多个实例，则至少您需要为要运行的每个节点分配独特的 Solr 主目录；理想情况下，每个主页都应位于不同的物理磁盘上，以便多个 Solr 节点在访问磁盘上的文件时不必相互竞争。拥有不同的 Solr 主目录意味着每个节点都需要一个不同的包含文件。而且，如果使用/etc/init.d/solr脚本来控制 Solr 作为服务，那么每个节点都需要单独的脚本。最简单的方法是使用服务安装脚本在同一主机上添加多个服务，例如：

```sh
sudo bash ./install_solr_service.sh solr-7.0.0.tgz -s solr2 -p 8984
```

上面显示的命令将 solr2 在端口 8984 上添加一个名为 running 的服务，使用 /var/solr2 可写（即“live”）文件；第二台服务器将仍然由 solr 用户拥有并运行，并将使用 /opt 其中的 Solr 分发文件。安装 solr2 服务之后，通过执行以下操作验证它是否正常工作：

```sh
sudo service solr2 restart
sudo service solr2 status
```
