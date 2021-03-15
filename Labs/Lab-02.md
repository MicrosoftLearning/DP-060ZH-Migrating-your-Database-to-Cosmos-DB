---
lab:
    title: '将 MongoDB 迁移到 Cosmos DB'
    module: '模块 2:将 MongoDB 工作负荷迁移到 Cosmos DB'
---

# 实验：将 MongoDB 迁移到 Cosmos DB

## 预计用时

90 分钟

## 场景

在本实验中，你将使用现有 MongoDB 数据库并将其迁移到 Cosmos DB。你将使用 Azure 数据库迁移服务。你还将了解如何将使用 MongoDB 数据库的现有应用程序重新配置为连接到 Cosmos DB 数据库。

本实验以从一系列 IoT 设备中捕获温度数据的示例系统为基础。温度和时间戳一起记录在 MongoDB 数据库中。每个设备都有唯一的 ID。你将运行一个模拟这些设备并将数据存储在数据库中的 MongoDB 应用程序。你还将使用支持用户查询每个设备的统计信息的第二个应用程序。将数据库从 MongoDB 迁移到 Cosmos DB 之后，你将这两个应用程序配置为连接到 Cosmos DB，并验证其是否仍正常运行。

本实验使用 Azure Cloud Shell 和 Azure 门户运行。

## 目标

你将在本实验中了解以下内容：

* 创建迁移项目
* 定义迁移的源和目标
* 执行迁移
* 验证迁移

## 注意

有关本实验的完整说明，请访问：https://github.com/MicrosoftLearning/DP-060ZH-Migrating-your-Database-to-Cosmos-DB/blob/master/Labs/Lab%202%20-%20Migrate%20MongoDB%20Workloads%20to%20Cosmos%20DB.md.

本实验的所有文件都位于 https://github.com/MicrosoftLearning/DP-060ZH-Migrating-your-Database-to-Cosmos-DB.

学生应先下载实验文件的副本，然后再执行实验。

## 实验练习

* 创建迁移项目
* 定义源和目标
* 执行迁移
* 验证迁移

## 实验回顾

你在本实验中使用了现有 MongoDB 数据库并将其迁移到了 Cosmos DB。你使用了 Azure 数据库迁移服务。你还了解了如何将使用此 MongoDB 数据库的现有应用程序重新配置为连接到 Cosmos DB 数据库。
