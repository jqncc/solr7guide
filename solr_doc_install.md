# 安装Solr

## 环境要求

操作系统: Linux，MacOS / OS X 和 Microsoft Windows。  
Java环境: jre1.8+

## 安装

在Unix兼容或Windows服务器上安装Solr，通常只需简单地提取（或解压缩）下载包。 

### 可用的Solr软件包

Solr可从Solr网站获取。您可以在此下载最新版本的(当前最新版是8.0)：<https://lucene.apache.org/solr/mirrors-solr-latest-redir.html>  
7.0下载地址:<http://archive.apache.org/dist/lucene/solr/7.0.0/>  
Solr有三个独立的软件包:

* solr-7.0.0.tgz：适用于Linux/Unix/OSX 系统
* solr-7.0.0.zip：适用于Windows 系统
* solr-7.0.0-src.tgz：Solr源代码包

### 准备安装Solr

Solr的安装非常简单,下载安装包,解压即可使用。但使用在生产环境不推荐使用默认配置,应根据实际环境对配置做出修改,比如端口号等.

### Solr的目录结构

**bin** 此目录中包含几个重要的脚本
* solr和solr.cmd  
    solr的控制脚本，bin/solr（* nix）或者bin/solr.cmd（ Windows）。这个脚本是启动和停止Solr的首选工具。您也可以在运行SolrCloud模式时创建集合或内核、配置身份验证以及配置文件。  
* post  
    Post Tool，它提供了用于发布内容到 Solr 的一个简单的命令行界面。
* solr.in  
     这些分别是为 * nix 和 Windows 系统提供的属性文件。在这里配置了Java、Jetty 和 Solr 的系统级属性。许多这些设置可以在使用bin/solr或者bin/solr.cmd时被覆盖，但这允许您在一个地方设置所有的属性。
* install_solr_services.sh  
     该脚本用于 * nix 系统以安装 Solr 作为服务。在 “将Solr用于生产 ” 一节中有更详细的描述。

**contrib**  该目录包含Solr专用功能的附加插件。  
**dist**  该目录包含主要的Solr jar文件。  
**docs**  solr文档  
**example**  包括演示各种Solr功能的几种类型的示例。 
**licenses**  该目录包括 Solr 使用的第三方库的所有许可证。  
**server**  此目录是Solr应用程序的核心所在。此目录中的 README 提供了详细的概述，但以下是一些特点：

* Solr 的web管理控制台（server/solr-webapp）
* Jetty 库(server/lib)
* 日志文件（server/logs）和日志配置（server/resources）。
* 有关如何自定义 Solr 的默认日志记录的详细信息，请参阅配置日志记录一节。
* 示例配置（server/solr/configsets）

---

### Solr示例

Solr 包括许多在开始时使用的示例文档和配置。如果您运行了 Solr 教程，您已经与这些文件中的某些文件进行了互动。
以下是 Solr 包含的示例：
* exampledocs  
   这是一系列简单的 CSV、XML 和 JSON 文件，可以在首次使用 Solr 时使用bin/post。有关和这些文件一起使用bin/post的更多信息，请参阅 Post 工具。
* example-DIH  
   此目录包含一些 DataImport Handler（DIH）示例，可帮助您开始在数据库、电子邮件服务器甚至 Atom 提要中导入结构化内容。每个示例将索引不同的数据集；有关这些示例的更多详细信息，请参阅 README。
* files  
    该目录为您提供了一个基本的搜索 UI，可以用于文档（例如 Word 或 PDF），您可能已经存储在本地。有关如何使用此示例的详细信息，请参阅README。
* films  
    该目录包含一组关于电影的强大数据，包括三种格式：CSV、XML 和 JSON。有关如何使用此数据集的详细信息，请参阅 README。
---

### Solr启动

启动脚本:bin/solr
启动命令:

```sh
solr start
```

这将在后台启动 Solr，默认监听端口为8983。
当您在后台启动Sol 时，脚本将等待确认 Solr 在正确启动后再返回到命令行提示符。 Solr命令的所有选项请查看 "[Solr控制脚本参考](solr_doc-m13y2fs3.md)"一节。

### 使用示例启动

Solr 还提供了一些有用的例子来帮助您了解主要功能。您可以使用该 -e 标志启动这些示例。例如，要启动 "techproducts" 示例，您可以执行以下操作:

```sh
bin/solr -e techproduct
```

目前，您可以运行的可用示例是：techproducts、dih、schemaless 和 cloud。有关每个示例的详细信息，请参阅运行示例配置一节。

### 检查Solr运行状态

如果您不确定Solr是否正常运行，则可以使用 status 命令：

```sh
bin/solr status
```

命令查看当前运行的Solr实例的基本信息，如版本和内存使用情况。  
另外还可以通过Solr web管理控制台查看,管理控制台默认地址：http://localhost:8983/solr/
![Solr管理界面](http://lucene.apache.org/solr/guide/7_0/images/running-solr/SolrAdminDashboard.png)
如果Solr 未运行，您的浏览器将提示无法连接到服务器。

### 创建核心

如果您没有使用示例配置启动 Solr，则需要创建一个核心才能进行索引和搜索。您可以运行以下操作：

```sh
bin/solr create -c
```

这将创建一个使用数据驱动模式的核心，当您将文档添加到索引时，该模式会尝试猜测正确的字段类型。
要查看创建新核心的所有可用选项，请执行以下操作：

```sh
bin/solr create -help
```
