---
lab:
    title: '将 Cassandra 迁移到 Cosmos DB'
    module: '模块 3:将 Cassandra 工作负荷迁移到 Cosmos DB'
---

# 实验：将 Cassandra 迁移到 Cosmos DB

## 预计用时

90 分钟

## 场景

在本实验中，你会将两个数据集从 Cassandra 迁移到 Cosmos DB。你将通过两种方式来迁移数据。首先，你将数据从 Cassandra 导出，并使用 CQLSH COPY 命令将数据库导入 Cosmos DB。然后，你将使用 Spark 迁移数据。你将运行查询原始 Cassandra 数据库中保存的数据的应用程序，然后将该应用程序重新配置为连接到 Cosmos DB，从而验证迁移是否成功。运行重新配置的应用程序的结果应保持不变。

本实验的场景与电子商务系统有关。客户可以订购产品。客户和订单详细信息都记录在 Cassandra 数据库中。你有一个生成摘要（例如，客户的订单列表、特定产品的订单，以及已购订单数量等各种汇总等）的应用程序。

本实验使用 Azure Cloud Shell 和 Azure 门户运行。

## 目标

你将在本实验中了解以下内容：

* 导出架构
* 使用 CQLSH COPY 迁移数据
* 使用 Spark 迁移数据
* 验证迁移

## 注意

有关本实验的完整说明，请访问：https://github.com/MicrosoftLearning/DP-060ZH-Migrating-your-Database-to-Cosmos-DB/blob/master/Labs/Lab%203%20-%20Migrate%20Cassandra%20Workloads%20to%20Cosmos%20DB.md

本实验的所有文件都位于 https://github.com/MicrosoftLearning/DP-060ZH-Migrating-your-Database-to-Cosmos-DB

学生应先下载实验文件的副本，然后再执行实验。

## 实验练习

* 导出架构
* 使用 CQLSH COPY 迁移数据
* 使用 Spark 迁移数据
* 验证迁移

## 实验回顾

在本实验中，你将两个数据集从 Cassandra 迁移到了 Cosmos DB。你通过两种方式迁移了数据。首先，你从 Cassandra 导出了数据，并使用 CQLSH COPY 命令将数据库导入了 Cosmos DB。然后，你使用 Spark 迁移了数据。你通过运行查询原始 Cassandra 数据库中保存的数据的应用程序，然后将该应用程序重新配置为连接到 Cosmos DB，验证了迁移是否成功。
