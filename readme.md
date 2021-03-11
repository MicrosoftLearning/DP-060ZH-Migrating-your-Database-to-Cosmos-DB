﻿# 将数据库迁移到 Cosmos DB

该存储库包含用于研讨会 **《将数据库迁移到 Cosmos DB》** 的练习的实验文档和源代码

## 实验 2：将 MongoDB 迁移到 Cosmos DB

在本实验中，你将使用现有 MongoDB 数据库并将其迁移到 Cosmos DB。你将使用 Azure 数据库迁移服务。你还将了解如何将使用 MongoDB 数据库的现有应用程序重新配置为连接到 Cosmos DB 数据库。

本实验以从一系列 IoT 设备中捕获温度数据的示例系统为基础。温度和时间戳一起记录在 MongoDB 数据库中。每个设备都有唯一的 ID。你将运行一个模拟这些设备并将数据存储在数据库中的 MongoDB 应用程序。你还将使用支持用户查询每个设备的统计信息的第二个应用程序。将数据库从 MongoDB 迁移到 Cosmos DB 之后，你将这两个应用程序配置为连接到 Cosmos DB，并验证其是否仍正常运行。

## 实验 3：将 Cassandra 迁移到 Cosmos DB

在本实验中，你会将两个数据集从 Cassandra 迁移到 Cosmos DB。你将通过两种方式来迁移数据。首先，你将数据从 Cassandra 导出，并使用 CQLSH COPY 命令将数据库导入 Cosmos DB。然后，你将使用 Spark 迁移数据。你将运行查询原始 Cassandra 数据库中保存的数据的应用程序，然后将该应用程序重新配置为连接到 Cosmos DB，从而验证迁移是否成功。运行重新配置的应用程序的结果应保持不变。

本实验的场景与电子商务系统有关。客户可以订购产品。客户和订单详细信息都记录在 Cassandra 数据库中。你有一个生成摘要（例如，客户的订单列表、特定产品的订单，以及已购订单数量等各种汇总等）的应用程序。

这两个实验都使用 Azure Cloud Shell 和 Azure 门户运行。
