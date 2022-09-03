---
title: NewSQL主流数据库对比——GIS篇
categories: 
- 数据库
tags:
- GIS
- NewSQL
date: 2021-08-08

---

# NewSQL主流数据库对比——GIS篇

[TOC]

## NewSQL 简介

NewSQL 是一种新方式关系数据库，意在整合 RDBMS 所提供的ACID事务特性，以及 NoSQL 提供的横向可扩展性。

大多数 NewSQL 数据库做了全新的设计，或是主要聚焦于 OLTP，或是采用了 OLTP/OLAP 的混合架构载的全新设计。

- 一致性：相对于可用性而言，NewSQL 更重视一致性，即侧重 CAP 中的 C 和 P。
- 内存数据库：一些 NewSQL 解决方案使用内存（RAM）作为存储介质。
- HTAP：HTAP（混合事务 / 分析处理，Hybrid Transactional/Analytical Processing）

![1166059074](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/1166059074.png)

## ！！！对比分析

| NewSQL                | 开发商                 | 是否开源              | 特性                                                         | 分布式                | 空间支持           | 社区活跃度                                                   | 拓展    |
| --------------------- | ---------------------- | --------------------- | ------------------------------------------------------------ | --------------------- | ------------------ | ------------------------------------------------------------ | ------- |
| Spanner               | 谷歌                   | ×                     | 存储的数据是Key-value                                        | 水平扩展              | ×                  | Google社区                                                   |         |
| OceanBase             | 阿里                   | 开源                  | MySQL/ORACLE兼容、增量数据放在内存、基线数据放在SSD盘、读写分 | 线性扩展              | ×                  | 阿里社区、博客                                               |         |
| TDSQL                 | 腾讯                   | ×                     | 提供两种版本MySQL和PostgreSQL                                | 水平扩展              | PostGIS            | 腾讯社区                                                     |         |
| Azure Cosmos DB       | 微软                   | 商业（2RMB/月）       | 支持键值、列存储、文档和图数据库，并支持通过 SQL 和 NoSQL API 提供数据。 | 水平扩展              | 支持               | 微软社区                                                     |         |
| TiDB                  | PingCAP                | 开源                  | 兼容 MySQL、TiDB、TiKV计算层与存储层的分离解耦架构           | 水平扩展              | ×                  | 社区、博客                                                   | TiSpark |
| SequoiaDB巨杉数据库   | 巨杉数据库             | 开源                  | 国产、金融级分布式关系型数据库                               | 集群                  | ×                  | 社区、博客                                                   |         |
| CockroachDB蟑螂数据库 | 蟑螂实验室             | 商业来源许可证（BSL） | 支持PostgreSQL、去中心化架构、支持kuberneter编排             | 水平扩展              | 部分兼容PostGIS    | Cockroach Labs 博客                                          |         |
| Citus                 | 微软并购               | GPL v3                | 开源 PostgreSQL **扩展**、并行处理、微软云、简单易用         | 支持分布式 PostgreSQL | PostGIS            | 论坛、博客                                                   |         |
| ClustrixDB            | MariaDB 2018年收购     | 收费                  | 类MYSQL                                                      | 分布式                |                    |                                                              |         |
| MemSQL/SingleStore    | 前 Facebook 工程师创办 | 开源的社区版          | **内存数据库**、兼容MySQL、MySQL InnoDB                      | 分布式                | 地理空间和全文搜索 | ![image-20210704154249267](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704154249267.png)社区论坛 |         |
| VoltDB                | Postgres和Ingres联合   | GPLv3、AGPL           | **内存数据库**、采用类MPP架构、并行的单线程处理方式确保数据一致性 | 水平扩展              | 仅支持简单空间查询 | 博客                                                         |         |
| Vitess                | CNCF                   | 开源                  | 部署，扩展和管理MySQL实例、支持kuberneter编排                | 无限水平扩展          | MySQL Spatial      | 博客、社区                                                   |         |
| NuoDB                 | NuoDB                  | ×                     | 去集中化、唯一一个具有专利的、弹性可伸缩的SQL关系数据库      |                       |                    | 文档都是视频                                                 |         |
| Altibase              |                        |                       | Hybrid DBMS                                                  |                       |                    |                                                              |         |
| YugabyteDB            |                        |                       |                                                              |                       |                    |                                                              |         |
| GenieDB               |                        |                       |                                                              |                       |                    |                                                              |         |
| ScaleDB               |                        |                       |                                                              |                       |                    |                                                              |         |
|                       |                        |                       |                                                              |                       |                    |                                                              |         |
|                       |                        |                       |                                                              |                       |                    |                                                              |         |





### [Azure Cosmos DB](https://docs.microsoft.com/zh-cn/azure/cosmos-db/use-cases)

> Azure Cosmos DB 是一种用于现代应用开发的完全托管式 NoSQL 数据库服务。 获得有保证的个位数毫秒级响应时间和 由 SLA 支持的 99.999% 可用性、 自动、即时的可伸缩性 ，以及用于 MongoDB 和 Cassandra 的开放源代码 API。
>
> 使用 Azure Synapse Link for Azure Cosmos DB 通过非 ETL 分析从实时数据中获取见解。

#### NoSQL 数据库与关系数据库之间的差别

- SQL API
- Cassandra API
  - Azure Cosmos DB Cassandra API 可以充当为 Apache Cassandra 编写的应用的**列族数据存储**。
  - 通过 Cassandra API 可以使用 Cassandra 查询语言 (CQL)、基于 Cassandra 的工具（如 cqlsh）和熟悉的 Cassandra 客户端驱动程序与 Azure Cosmos DB 中存储的数据进行交互。
- Gremlin API
  - Azure Cosmos DB 通过 Gremlin API 在为任何规模设计的完全托管数据库服务中提供**图形数据库**服务。
  -  使用 Gremlin 查询语言。
- 表 API
  - 类似于CloudTable。
  - Azure Table Storage  提供的表 API 适用于为 Azure **表存储**。
  - 表 API 包含可用于 .NET、Java、Python 和 Node.js 的客户端 SDK。
- Azure Cosmos DB API for MongoDB
  - 通过用于 MongoDB 的 Azure Cosmos DB API，可以轻松使用 Cosmos DB，就像它是 MongoDB **文档数据库**一样。 

#### Azure Cosmos DB 中的地理空间和 GeoJSON 位置数据

- 支持两种空间数据类型：geometry 数据类型和 geography 数据类型 。

  - geometry 类型在欧几里得（平面）坐标系统中表示数据
  - **Geography** 类型表示圆形地球坐标系中的数据。

- 支持的空间数据类型

  - Point
  - LineString
  - Polygon
  - MultiPolygon

- 地理空间数据查询

  ![image-20210704152452747](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704152452747.png)

- 地理空间数据编制索引

  - 简单来说，测地坐标的几何图形会投影在 2D 平面上，并使用 **四叉树** 以渐进方式划分成单元格。 这些单元格会根据 **Hilbert 空间填充曲线** 内的单元格位置映射到 1D，并保留点的位置。 

- https://docs.microsoft.com/zh-cn/azure/cosmos-db/sql-query-geospatial-intro

- https://docs.microsoft.com/zh-cn/azure/cosmos-db/sql-query-geospatial-query



### [Altibase（商用收费）](http://cn.altibase.com/)



### [CockroachDB](https://www.cockroachlabs.com/)

> CockroachDB（https://www.cockroachlabs.com）是Google备受瞩目的Spanner的开源模仿，承诺提供一种高存活性、强一致性，可横向扩展的SQL数据库。主要的设计目标是全球一致性和可靠性，从蟑螂（cockroach）的命名上是就能看出这点 [ 打不死的小强：) ]。Cockroach节点是均衡的，其设计目标是同质部署（只有一个二进制包）且最小配置。CockroachDB的扩展非常容易，只要一行命令，秒级进行。
>

- PostgreSQL生态中的很多工具、程序和应用能够适用于CockroachDB(不用修改或少量修改)。
- 虽然CockroachDB支持PostgreSQL语法和驱动程序，但它不是完全兼容PostgreSQL。
- 2020年的20.2版本开始支持[geospatial](https://www.cockroachlabs.com/blog/spatial-data/)，但未完全兼容PostGIS。
- 采用分层架构，最顶层是SQL层。SQL层构建于分布式KV存储之上

![img](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/20190413215412940.png)

https://blog.csdn.net/qq_34924156/article/details/89236693

- 用户手册：http://doc.cockroachchina.baidu.com/
- 社区：https://www.cockroachlabs.com/blog/



### [Citus](http://citusdb.cn/)

> CitusDB采用PostgreSQL的插件形式(not a fork)，即享受PostgreSQL的强大支持，又同时拥有分布式数据库能力
>
> CitusDB 与 HDFS 的分布式非常相似，在 Master 上存储元数据，Work 节点存储分片，同时 1 个分片至少要存储在 2 个 Work 节点（可配置更多）上保障其可用性。

![image-20210703221436470](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210703221436470.png)

- 苏宁citus分布式数据库应用实践

https://itdks.su.bcebos.com/844d50a9b73f4d8283d746cbff23b5e9.pdf

- 利用 citus 支持地理大数据。全世界的点状POI，数据量级已达亿级别，存储在单一的PostgreSQL数据表中，若查询涉及到全表扫描，性能会差很多

https://www.giserdqy.com/secdev/openlayers/21870/

- `Citus`是一个PostgreSQL的扩展，类似于PostGIS、pgRouting，它主要的作用是将一张大表进行水平分表，每个分表称为“分片（Shard）”，按照一定的规则分布(?)到多台机器上，并且将查询分布到不同节点并行执行，最后汇总结果。

![img](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/TUIK%7ES14Z%606A_I%60E%7BFPYT6L.png)

- 论坛：http://citusdb.cn/?post_type=forum
- 博客：http://citusdb.cn/?cat=13
- 手册：http://docs.citusdata.com/en/v10.0/



### [ClustrixDB(收费)](https://mariadb.com/products/clustrixdb/)







### [MemSQL](https://www.memsql.com/software/)

> 由前 Facebook 工程师创办的 MemSQL，号称世界上最快的分布式关系型数据库，兼容 MySQL 但快30倍，能实现每秒 150 万次事务。原理是仅用内存并将 SQL 预编译为C++。
>

- MemSQL is Now SingleStore
- 2018年，MemSQL 搞了个炸裂消息，[宣布](https://link.zhihu.com/?target=https%3A//www.memsql.com/blog/memsql67/) MemSQL 6.7 可以免费用于生产环境，没有功能限制，磁盘容量无限制，只限制整个 MemSQL 集群总内存使用不得超过 128 GB，如果做双机备份，那么就是每台机器 64 GB，已经很够用了。
- 使用ACID事务每秒接收数百万个事件，同时以关系SQL、JSON、地理空间和全文搜索格式分析数十亿行数据。

![image-20210704150733553](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704150733553.png)

- 地理空间支持

  - SingleStore使用了一个类似于googleearth的球形模型。它假设一个半径为6367444.66米的完美球形地球。

  - 地理数据类型

    ![image-20210704151619023](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704151619023.png)

  - 空间查询

    ![image-20210704151741992](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704151741992.png)

  - GEOJSON。SingleStore 没有本机 GeoJSON 支持。但是，我们有通用的 JSON 支持以及计算列。结合这些功能，您可以使用 SingleStore 的内置 JSON 类型和 GEOGRAPHYPOINT 类型导入 GeoJSON 数据的子集。

    ![image-20210704151949000](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704151949000.png)

- 社区网址：https://www.singlestore.com/community-hub/
- 资源手册：https://www.singlestore.com/resources/

### [NuoDB](https://nuodb.com/)

> NuoDB 是newsql数据库的典型代表之一，同时也是世界上首个也是唯一一个具有专利的、弹性可伸缩的SQL关系数据库，主要用于去集中化的计算资源。



### [OceanBase](https://open.oceanbase.com/)

> OceanBase 社区版是一款开源分布式 HTAP（Hybrid Transactional/Analytical Processing）数据库管理系统，具有原生分布式架构，支持金融级高可用、透明水平扩展、分布式事务、多租户和语法兼容等企业级特性。

> 2020 年 5 月，OceanBase 以 7.07亿 tpmC 的在线事务处理性能，打破了 OceanBase 自己在 2019 年创造的 6088万 tpmC 的 TPC-C 世界纪录。截止至目前，OceanBase是第一个也是唯一一个上榜的中国数据库。

![image-20210704164251964](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704164251964.png)

- 版本选择

  ![image-20210704164312198](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704164312198.png)

- 产品优势

  - **高性能**：OceanBase采用了读写分离的架构，把数据分为基线数据和增量数据。其中增量数据放在内存里（MemTable），基线数据放在SSD盘（SSTable）。对数据的修改都是增量数据，只写内存。所以DML是完全的内存操作，性能非常高。
  - **低成本**：OceanBase通过数据编码压缩技术实现高压缩。数据编码是基于数据库关系表中不同字段的值域和类型信息，所产生的一系列的编码方式，它比通用的压缩算法更懂数据，从而能够实现更高的压缩效率。
  - **高兼容**：兼容常用MySQL/ORACLE功能及MySQL/ORACLE前后台协议，业务零修改或少量修改即可从MySQL/ORACLE迁移至OceanBase。
  - **高可用**：数据采用多副本存储，少数副本故障不影响数据可用性。通过“三地五中心”部署实现城市级故障自动无损容灾。

- 定价

  ![image-20210704164141515](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704164141515.png)

- 社区：https://open.oceanbase.com/community/contribution

- 博客：https://open.oceanbase.com/articles
- 文档：https://open.oceanbase.com/docs



### SequoiaDB



### [TDSQL](https://intl.cloud.tencent.com/zh/product/dcdb)

> 分布式数据库（Tencent Distributed SQL，以下简称 TDSQL）是腾讯打造的一款企业级数据库产品，具备强一致高可用、全球部署架构、高 SQL 兼容度、分布式水平扩展、高性能、完整的分布式事务支持、企业级安全等特性，同时提供智能 DBA、自动化运营、监控告警等配套设施，为客户提供完整的分布式数据库解决方案。
>
> 稳定、安全、高性能的分布式数据库，兼容 PostgreSQL 和 MySQL。

1. MySQL 版

   高度兼容 MySQL，支持水平拆分（分表）的高性能数据库。部署在腾讯云上的一种支持自动水平拆分、Shared Nothing 架构的分布式数据库。TDSQL MySQL版 默认部署主备架构，提供容灾、备份、恢复、监控、迁移等全套解决方案，适用于 TB 或 PB 级的海量数据库场景。



2. PostgreSQL 版 （原 TBase）

   稳定、安全、高性能的分布式数据库服务，满足您海量 HTAP 场景。集高扩展性、高SQL兼容度、完整的分布式事务支持、多级容灾能力以及多维度资源隔离等能力于一身。TDSQL PostgreSQL 版采用无共享的集群架构，为用户提供容灾、备份、恢复、监控、安全、审计等全套解决方案，适用于GB～PB级的海量 HTAP 场景。  

    具有丰富的周边生态：

   - 支持强大的地理信息系统（GIS）。通过集群化的 PostGis 插件，支持存储空间地理数据，使 TDSQL PostgreSQL版 成为一个空间数据库，能够通过 SQL 语言高效的进行空间数据管理、数量测量和几何拓扑分析。
   - TDSQL PostgreSQL版 不仅是一个分布式关系型数据库系统，同时还支持非关系数据类型 JSON。
   - 支持 Foreign Data Wrappers（FDW）功能，该功能实现了部分的 SQL/MED 规定，允许用户使用普通 SQL 查询来访问位于 PostgreSQL 之外的数据。
     FDW 功能提供一套编程接口，用户可进行插件式的二次开发，建立外部数据源和数据库间的数据通道。大多数情况下用户可用 oracle_fdw、mysql_fdw、postgres_fdw，非关系型数据库的 redis_fdw、mongodb_fdw，以及大数据的 hive_fdw、hdfs_fdw 等。基于 FDW 功能和已有插件，TDSQL PostgreSQL版 提供强大的数据库联邦能力，通过 TDSQL PostgreSQL版 能够访问已有的多个数据源的数据。

![image-20210704180213145](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704180213145.png)

- 价格

  ​	![image-20210704180327148](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210704180327148.png)



### [TiDB ](https://pingcap.com/)



### [VoltDB](https://www.voltdb.com/)



### [Vitess](https://vitess.io/zh/)

![image-20210703213038339](https://raw.githubusercontent.com/2757412961/TyporaImageRepository/master/img/image-20210703213038339.png)

- Vitess包括使用与本机查询协议兼容的JDBC和Go数据库驱动。自2011年以来，Vitess一直为YouTube所有的数据库提供服务，现在已被许多企业采用并应用于实际生产。

- Vitess是一个用于部署、扩展和管理大型MySQL实例集群的数据库解决方案。Vitess集Mysql数据库的很多重要特性和NoSQL数据库的可扩展性于一体。它的架构设计使得您可以像在物理机上一样在公共云或私有云架构中有效运行。它结合并扩展了许多重要的MySQL功能，同时兼具NoSQL数据库的可扩展性。

- 一种云原生技术 Cloud-native
  - 容器化（Containerized）：每个部分（应用程序，进程等）都封装在自己的容器中。这有助于重复性，透明度和资源隔离。
  - 动态编排（Dynamically orchestrated）：动态的调度和管理容器以优化资源利用。
  - 面向微服务（Microservices oriented）：应用程序基于微服务架构，显著提高架构演进的灵活性和可维护性。



**CNCF案例研究：京东如何使用Vitess管理超大规模数据库**

https://cloud.tencent.com/developer/article/1548244

















# NewSQL，NoSQL与OldSQL的混合部署

- OldSQL+NewSQL
- OldSQL+NoSQL
- NewSQL+NoSQL

https://supmarket.net/xy/771126462