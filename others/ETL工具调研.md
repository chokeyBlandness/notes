# ETL工具调研

ETL（Extract-Transform-Load）是将业务系统的数据经过抽取、清洗转换之后加载到数据仓库的过程，目的是将企业中的分散、零乱、标准不统一的数据整合到一起，为企业的决策提供分析依据。ETL是BI项目重要的一个环节。 通常情况下，在BI项目中ETL会花掉整个项目至少1/3的时间,ETL设计的好坏直接关接到BI项目的成败。    

## DataX

alibaba开源的只到0.0.1-SNAPSHOT版本，GitHub上个人开发的datax-web也是基于此来扩展的。

GitHub地址：https://github.com/alibaba/DataX。

https://datax.readthedocs.io/zh_CN/3.2.5/。此文档描述的DataX3属于个人基于alibaba开源的版本进行扩展开发出来的，star和fork都比较少。目前不知道此项目和alibaba所说的DataX有什么关系。GitHub地址：https://github.com/wgzhao/DataX。

## Kettle

Kettle是一款国外开源的ETL工具，目前为Pentaho的一个组件，纯java编写，可以在Window、Linux、Unix上运行，数据抽取高效稳定。

GitHub地址：https://github.com/pentaho/pentaho-kettle。

主要的组件：

| **名称**    | **描述**                                                     |
| ----------- | ------------------------------------------------------------ |
| **Spoon**   | 通过图形接口，用于编辑作业和转换的桌面应用。                 |
| **Pan**     | 允许你批量运行由Spoon设计的ETL转换 (例如使用一个时间调度器)。Pan是一个后台执行的程序，没有图形界面。 |
| **Kitchen** | 允许你批量使用由Chef设计的任务 (例如使用一个时间调度器)。KITCHEN也是一个后台运行的程序。 |
| **Chef**    | 允许你创建任务（Job）。任务通过允许每个转换，任务，脚本等等，更有利于自动化更新数据仓库的复杂工作。任务通过允许每个转换，任务，脚本等等。任务将会被检查，看看是否正确地运行了。 |

数据源连接方式：Native(JDBC)、ODBC、JNDI【、OCI只针对Oracle DB】。

支持的数据库类型：AS/400、Apache Derby、Borland Interbase、Calpont InfiniDB、Cloudera Impala、Exasol 4、ExtenDB、Firebird SQL、Generic database、Google BigQuery、Greenplum、Gupta SQL Base、H2、Hadoop Hive、Hadoop Hive 2/3、Hive Warehouse Connector、Hypersonic、IBM DB2、Impala、Infobright、Informix、Ingres、Ingres VectorWise、Intersystems Cache、KingbaseES、LucidDB、MS Access、MS SQL Server、MariaDb、MariaDB(SAP DB)、MonetDB、MySQL、Native Mondrian、Neoview、Netezza、Oracle、Oracle RDB、Palo MOLAP Server、Pentaho Data Services、PostgreSQL、Redshift、Remedy Action Request System、SQLite、Snowflake、SparkSQL、Sybase、SybaseIQ、Teradata、Universe database、Vertica、Vertica 5+、dBase III,IV or 5。

可自定义扩展接口，官网有案例：https://sourceforge.net/projects/pentaho/files/Pentaho%209.1/plugins/。

#### DataX和Kettle对比

- Kettle拥有自己的管理控制台，可以直接在客户端进行etl任务制定，不过是CS架构，而不支持BS浏览器模式，体积也比较大。DataX并没有界面，界面完全需要自己开发，增加了很大工作量。
- Kettle可以与我们自己的工程进行集成，通过JAVA代码集成即可，可以在java中调用kettle的转换、执行、结束等动作，这个还是有意义的，而DataX是不支持的，DataX是以执行脚本的方式运行任务的，当然完全吃透源码的情况下，应该也是可以调用的。
- Kettle已经加入BI组织Pentaho，加入后kettle的开发粒度和被关注度更进一步提升。DataX开源的支持粒度不高，关注度远没有kettle高，代码提交次数更是少的很。
- 根据网上参考信息，网友测试 kettle全量抽取较大数据量时，抽取时间长，对比测试 datax比kettle快。
- DataX脚本运行需要依赖python环境，Kettle只需要Java环境。
- Kettle可集群部署，DataX只存在单机模式。

### 其他

其他还有DataPipeline、Talend、Informatica、Goldengate、datastage。但是都没找到开源的内容。https://cloud.tencent.com/developer/article/1531141。