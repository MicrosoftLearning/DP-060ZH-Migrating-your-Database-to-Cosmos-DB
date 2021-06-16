- [实验室 3：将 Cassandra 工作负载迁移到 Cosmos DB](#lab-3-migrate-cassandra-workloads-to-cosmos-db)
  - [练习 1：设置](#exercise-1-setup)
    - [任务 1：创建资源组和虚拟网络](#task-1-create-a-resource-group-and-virtual-network)
    - [任务 2：创建 Cassandra 数据库服务器](#task-2-create-a-cassandra-database-server)
    - [任务 3：填充 Cassandra 数据库](#task-3-populate-the-cassandra-database)
  - [练习2：使用 CQLSH COPY 命令将数据从 Cassandra 迁移到 Cosmos DB](#exercise-2-migrate-data-from-cassandra-to-cosmos-db-using-the-cqlsh-copy-command)
    - [任务 1：创建 Cosmos 帐户和数据库](#task-1-create-a-cosmos-account-and-database)
    - [任务 2：从 Cassandra 数据库导出数据](#task-2-export-the-data-from-the-cassandra-database)
    - [任务 3：将数据导入 Cosmos DB](#task-3-import-the-data-to-cosmos-db)
    - [任务 4：验证数据迁移是否成功](#task-4-verify-that-data-migration-was-successful)
    - [任务 5：知识总结](#task-5-clean-up)
  - [练习 3：使用 Spark 将数据从 Cassandra 迁移到 Cosmos DB](#exercise-3-migrate-data-from-cassandra-to-cosmos-db-using-spark)
    - [任务 1：创建 Spark 群集](#task-1-create-a-spark-cluster)
    - [任务 2：创建用于迁移数据的笔记本](#task-2-create-a-notebook-for-migrating-data)
    - [任务 3：连接 Cosmos DB 并创建表](#task-3-connect-to-cosmos-db-and-create-tables)
    - [任务 4：连接 Cassandra 数据库并检索数据](#task-4-connect-to-the-cassandra-database-and-retrieve-data)
    - [任务 5：将数据插入 Cosmos DB 表并运行笔记本](#task-5-insert-data-into-cosmos-db-tables-and-run-the-notebook)
    - [任务 6：验证数据迁移是否成功](#task-6-verify-that-data-migration-was-successful)
    - [任务 7：知识总结](#task-7-clean-up)

# 实验室 3：将 Cassandra 工作负载迁移到 Cosmos DB

在本实验中，你会将两个数据集从 Cassandra 迁移到 Cosmos DB。你将通过两种方式来迁移数据。首先，你将数据从 Cassandra 导出，并使用 **CQLSH COPY** 命令将数据库导入 Cosmos DB。然后，你将使用 Spark 迁移数据。你将通过对 Cosmos DB 数据库中保存的数据运行查询来验证迁移是否成功。 

本实验的场景与电子商务系统有关。客户可以订购产品。客户和订单详细信息都记录在 Cassandra 数据库中。 

本实验使用 Azure Cloud Shell 和 Azure 门户运行。

## 练习 1：设置

在第一个练习中，你将创建 Cassandra 数据库，以保存客户和订单数据。

### 任务 1：创建资源组和虚拟网络

1. 在 Internet 浏览器中，导航到 https://portal.azure.com 并登录。
1. 在 Azure 门户中，选择 **“资源组”**，然后选择 **“+ 添加”**。
1. 在 **“创建资源组页面”** 上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | cassandradbrg |
    | 区域 | 选择离你最近的位置 |

1. 选择 **“查看 + 创建”**。
1. 选择 **“创建”**，然后等待资源组创建完成。
1. 在 Azure 门户的左侧窗格中，选择 **“+ 创建资源”**。
1. 在 **“新建”** 页面的 **“搜索市场”** 框中，键入 **“虚拟网络”**，然后按 Enter。
1. 在 **“虚拟网络”** 页面上，选择 **“创建”**。
1. 在 **“基本信息”** 窗格上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | cassandradbrg |
    | 名称 | databasevnet |
    | 区域 | 选择为资源组指定的相同位置 |

1. 选择 **“下一页: IP 地址”**。
1. 在 **“IP 地址”** 页面上，将 **“IPv4 地址空间”** 设置为 **“10.0.0.0/24”**。
1. 选择默认子网，然后选择 **“删除子网”**。
1. 选择 **“+ 添加子网”**。在 **“添加子网”** 窗格中，将 **“子网名称”** 设置为 **“默认值”**，并将 **“子网地址范围”** 设置为 **“10.0.0.0/28”**，然后选择 **“添加”**。
1. 在 **“IP 地址”** 页面上，选择 **“下一页: 安全性”**。
1. 在 **“安全性”** 页面上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | BastionHost | 已禁用 |
    | DDoS 防护标准  | 已禁用 |
    | 防火墙 | 已禁用 |

1. 选择 **“查看 + 创建”**
1. 在 **“查看 + 创建”** 页面上，选择 **“创建”**，等待虚拟网络创建完成，然后再继续。

### 任务 2：创建 Cassandra 数据库服务器

1. 在 Azure 门户的左侧窗格中，选择 **“+ 创建资源”**。
1. 在 **“在市场中搜索”** 框中，键入 **“Cassandra 通过 Bitnami 认证”**，然后按 Enter。
1. 在 **“Cassandra 通过 Bitnami 认证”** 页面上，选择 **“创建”**。
1. 在 **“创建虚拟机”** 页面上，输入以下详细信息。

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | cassandradbrg |
    | 虚拟机名称 | cassandraserver |
    | 区域 | 选择为资源组指定的相同位置 |
    | 可用性选项 | 无需基础结构冗余 |
    | 映像 | Cassandra 通过 Bitnami 认证 - Gen1 |
    | 大小 | 标准 D2 v2 |
    | 身份验证类型 | 密码 |
    | 用户名 | azureuser |
    | 密码 | Pa55w.rdPa55w.rd |
    | 确认密码 | Pa55w.rdPa55w.rd |

1. 选择 **“下一步: 磁盘 \>”**。
1. 在 **“磁盘”** 页面上，将设置保留为默认值，然后选择 **“下一步: 网络 \>”**。
1. 在 **“网络”** 页面上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 虚拟网络 | databasevnet |
    | 子网 | 默认 (10.0.0.0/28) |
    | 公共 IP | （新）cassandraserver-ip |
    | NIC 网络安全组 | 高级 |
    | 配置网络安全组 | （新）cassandraserver-nsg |
    | 加速网络 | 关闭 |
    | 负载均衡 | 否 |

1. 选择 **“下一步: 管理 \>”**。
1. 在 **“管理”** 页面上，将设置保留为默认值，然后选择 **“下一步: 高级 \>”**。
1. 在 **“高级”** 页面上，将设置保留为默认值，然后选择 **“下一步: 标记 \>”**。
1. 在 **“标记”** 页面上，将设置保留为默认值，然后选择 **“下一步: 查看 + 创建 \>”**。
1. 在验证页面上，选择 **“创建”**。
1. 等待虚拟机创建完毕，然后继续。
1. 在 Azure 门户的左侧窗格中，选择 **“所有资源”**。
1. 在 **“所有资源”** 页面上，选择 **“cassandraserver-nsg”**。
1. 在 **“cassandraserver-nsg”** 页面上的 **“设置”** 下，选择 **“入站安全规则”**。
1. 在 **“cassandraserver-nsg - 入站安全规则”** 页面上，选择 **“+ 添加”**。
1. 在 **“添加入站安全规则”** 窗格中，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 源 | 任何 |
    | 源端口范围 | * |
    | 目标 | 任何 |
    | 目标端口范围 | 9042 |
    | 协议 | 任何 |
    | 操作 | 允许 |
    | 优先级 | 1020 |
    | 名称 | Cassandra-port |
    | 描述 | 客户端用于连接到 Cassandra 的端口 |

1. 选择 **“添加”**。

### 任务 3：填充 Cassandra 数据库

1. 在 Azure 门户的左侧窗格中，选择 **“所有资源”**。
1. 在 **“所有资源”** 页面上，选择 **“cassandraserver-ip”**。
1. 在 **“cassandraserver-ip”** 页面上，记下 **“IP 地址”**。
1. 在 Azure 门户顶部的工具栏中，选择 **“Cloud Shell”**。
1. 如果出现 **“你没有安装存储”** 消息框，请选择 **“创建存储”**。
1. Cloud Shell 启动时，在 Cloud Shell 窗口上方的下拉列表中，选择 **“Bash”**。
1. 在 Cloud Shell 中，如果尚未执行实验 2，请运行以下命令以下载此研讨会的示例代码和数据：

    ```bash
    git clone https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB migration-workshop-apps
    ```

1. 移至 **“migration-workshop-apps/Cassandra”** 文件夹：

    ```bash
    cd ~/migration-workshop-apps/Cassandra
    ```

1. 输入以下命令以将安装脚本和数据复制到 **cassandraserver** 虚拟机。请将 *“\<ip address\>”* 替换为 **“cassandraserver-ip”** IP 地址的值：

    ```bash
    scp *.* azureuser@<ip address>:~
    ```

1. 在提示符下，键入 **“是”** 以继续连接。
1. 在 **“密码”** 提示符下，输入密码 **“Pa55w.rdPa55w.rd”**
1. 键入以下命令连接 **cassandraserver** 虚拟机。请指定 **cassandraserver** 虚拟机的 IP 地址：

    ```bash
    ssh azureuser@<ip address>
    ```

1. 在 **“密码”** 提示符下，输入密码 **“Pa55w.rdPa55w.rd”**
1. 运行以下命令以连接到 Cassandra 数据库，创建本实验所需的表，然后填充它们。

    ```bash
    bash upload.sh
    ```

    该脚本创建两个分别名为 **“customerinfo”** 和 **“orderinfo”** 的密钥空间。该脚本在 **“customerinfo”** 密钥空间中创建了一个名为 **“customerdetails”** 的表，并在 **“orderinfo”** 密钥空间中创建了两个分别名为 **“orderdetails”** 和 **“orderline”** 的表。

1. 运行以下命令，并记下此文件中的默认密码：

    ```bash
    cat bitnami_credentials
    ```

1. 以用户 **cassandra**（这是在设置虚拟机时创建的默认 Cassandra 用户名）身份启动 Cassandra Query Shell。请将 *“\<password\>”* 替换为上一步骤中的默认密码：

    ```bash
    cqlsh -u cassandra -p <password>
    ```

1. 在 **“cassandra@cqlsh”** 提示符下，运行以下命令。此命令显示 customerinfo.customerdetails 表中的前 100 行：

    ```cqlsh
    select *
    from customerinfo.customerdetails
    limit 100;
    ```

    请注意，数据按 **stateprovince** 列进行聚类，然后按 **customerid** 进行排序。通过进行分组，应用程序可以快速查找位于同一区域的所有客户。

1. 请运行以下命令。此命令显示 **orderinfo.orderdetails** 表中的前 100 行：

    ```cqlsh
    select *
    from orderinfo.orderdetails
    limit 100;
    ```

    **“orderinfo.orderdetails”** 表包含每个客户下的订单列表。记录的数据包括下单的日期和订单的价值。数据按 **customerid** 列进行聚类，以便应用程序可以快速找到指定客户的所有订单。

1. 请运行以下命令。此命令显示 **orderinfo.orderline** 表中的前 100 行：

    ```cqlsh
    select *
    from orderinfo.orderline
    limit 100;
    ```

    此表包含每个订单的项目。数据按 **orderid** 列进行聚类，并按 **orderline** 进行排序。

1. 退出 Cassandra Query Shell：

    ```cqlsh
    exit;
    ```

1. 在 **“bitnami@cassandraserver”** 提示符下，键入以下命令以断开与 Cassandra 服务器的连接并返回 Cloud Shell：

    ```bash
    exit
    ```

## 练习2：使用 CQLSH COPY 命令将数据从 Cassandra 迁移到 Cosmos DB 

你现在已经创建并填充了 Cassandra 数据库。在本练习中，你将使用 Cassandra API 创建一个 Cosmos DB 帐户，然后将数据从 Cassandra 数据库迁移到 Cosmos DB 帐户中的数据库。

### 任务 1：创建 Cosmos 帐户和数据库

1. 返回 Azure 门户。
1. 在左窗格中，选择 **“+ 创建资源”**。
1. 在 **“新建”** 页面上的 **“在市场中搜索”** 框中，键入 **“Azure Cosmos DB”**，最后按 Enter。
1. 在 **“Azure Cosmos DB”** 页面上，选择 **“创建”**。
1. 在 **“创建 Azure Cosmos DB 帐户”** 页面上，输入以下设置：

    | 属性  | 值  |
    |---|---|
    | 订阅 | 选择你的订阅 |
    | 资源组 | cassandradbrg |
    | 帐户名 | cassandra*nnn*，其中 *“nnn”* 表示你选择的随机数字 |
    | API | Cassandra |
    | Notebooks | 关 |
    | 位置 | 指定用于 Cassandra 服务器和虚拟网络的相同位置 |
    | 容量模式 | 预配的吞吐量 |
    | 应用免费层折扣 | 应用 |
    | 帐户类型 | 非生产 |
    | 异地冗余 | 禁用 |
    | 多区域写入 | 禁用 |
    | 可用性区域 | 禁用 |

1. 选择 **“查看 + 创建”**。
1. 在验证页面上，选择 **“创建”**，然后等待 Cosmos DB 帐户完成部署。
1. 在左侧窗格中，选择 **“所有资源”**。
1. 在 **“所有资源”** 页面上，选择 Cosmos DB 帐户 (**cassandra*nnn***)。
1. 在 **“cassandra*nnn*”** 页面上，选择 **“数据资源管理器”**。
1. 在 **“数据资源管理器”** 窗格中，选择 **“新建表”**。
1. 在 **“添加表”** 窗格中，指定以下设置，然后选择 **“确定”**：

    | 属性  | 值  |
    |---|---|
    | 密钥空间名称 | 选择 **“新建”**，然后键入 **“customerinfo”** |
    | 预配密钥空间吞吐量 | 取消选中 |
    | 输入 tableId | customerdetails |
    | *“CREATE TABLE”* 框 | （customerid int、firstname text、lastname text、email text、stateprovince text、PRIMARY KEY ((stateprovince)、customerid)） |
    | 吞吐量 | 10000 |

1. 在 **“数据资源管理器”** 窗格中，选择 **“新建表”**。
1. 在 **“添加表”** 窗格中，指定以下设置：

    | 属性  | 值  |
    |---|---|
    | 密钥空间名称 | 选择“新建”，然后键入 **“orderinfo”** |
    | 预配密钥空间吞吐量 | 取消选中 |
    | 输入 tableId | orderdetails |
    | *“CREATE TABLE”* 框 | （orderid int、customerid int、orderdate date、ordervalue decimal、PRIMARY KEY ((customerid)、orderdate, orderid)） |
    | 吞吐量 | 10000 |

1. 选择 **“确定”**。
1. 在 **“数据资源管理器”** 窗格中，选择 **“新建表”**。
1. 在 **“添加表”** 窗格中，指定以下设置：

    | 属性  | 值  |
    |---|---|
    | 密钥空间名称 | 选择 **“使用现有项”**，然后选择 **“orderinfo”** |
    | 输入 tableId | orderline |
    | *“CREATE TABLE”* 框 | （orderid int、orderline int、productname text、quantity smallint、orderlinecost decimal、PRIMARY KEY ((orderid)、productname、orderline)） |
    | 吞吐量 | 10000 |

1. 选择 **“确定”**。

### 任务 2：从 Cassandra 数据库导出数据

1. 返回 Cloud Shell。
1. 运行以下命令以连接到 cassandra 服务器。请将 *\<ip address\>* 替换为虚拟机的 IP 地址。系统出现提示时，请输入密码 **“Pa55w.rdPa55w.rd”**：

    ```bash
    ssh azureuser@<ip address>
    ```

1. 启动 Cassandra Query Shell。请在 **bitnami_credentials** 文件中指定密码：

    ```bash
    cqlsh -u cassandra -p <password>
    ```

1. 在 **“cassandra@cqlsh”** 提示符下，运行以下命令。此命令将下载 **customerinfo.customerdetails** 表中的数据并将其写入名为 **“customerdata”** 的文件。该命令应导出 19119 个行：

    ```cqlsh
    copy customerinfo.customerdetails
    to 'customerdata';
    ```

1. 运行以下命令以将 **orderinfo.orderdetails** 表中的数据导出到名为 **“orderdata”** 的文件中。此命令应导出 31465 个行：

   ```cqlsh
    copy orderinfo.orderdetails
    to 'orderdata';
    ```

1. 运行以下命令以将 **orderinfo.orderline** 表中的数据导出到名为 **“orderline”** 的文件中。此命令应导出 121317 个行：

   ```cqlsh
    copy orderinfo.orderline
    to 'orderline';
    ```

1. 关闭 Cassandra Query Shell：

    ```cqlsh
    exit;
    ```

### 任务 3：将数据导入 Cosmos DB

1. 切换回 Azure 门户中的 Cosmos DB 帐户。
1. 在 **“设置”** 下，选择 **“连接字符串”**，并记下以下各项：

   - 接触点
   - 端口
   - 用户名
   - 主密码

1. 返回 Cassandra 服务器，然后启动 Cassandra Query Shell。这一次，连接到你的 Cosmos DB 帐户。请将 **cqlsh** 命令的参数替换为刚刚记下的值：

    ```bash
    export SSL_VERSION=TLSv1_2
    export SSL_VALIDATE=false

    cqlsh <contact point> <port> -u <username> -p <primary password> --ssl
    ```

    请注意，Cosmos DB 需要 SSL 连接。

1. 在 **“cqlsh”** 提示符下，运行以下命令以导入之前从 Cassandra 数据库导出的数据：

    ```cqlsh
    copy customerinfo.customerdetails from 'customerdata' with chunksize = 200;
    copy orderinfo.orderdetails from 'orderdata' with chunksize = 200;
    copy orderinfo.orderline from 'orderline' with chunksize = 100;
    ```

    这些命令应导入19119 个 **“customerdetails”** 行、31465 个 **“orderdetails”** 行和 121317 个 **“orderline”** 行。如果这些命令失败并显示超时错误，则可以通过以下方法解决此问题：
    - 增加 Azure 门户中相应表的吞吐量（并在之后再次降低吞吐量，以避免产生多余的吞吐量费用），或者
    - 减小每项复制操作的块大小。减小块大小会降低引入速率，而增加吞吐量则会增加成本。

### 任务 4：验证数据迁移是否成功

1. 返回到 Azure 门户中的 Cosmos DB 帐户，然后选择 **“数据资源管理器”**。
1. 在 **“数据资源管理器”** 窗格中，依次展开 **“customerinfo”** 密钥空间、**“customerdetails”** 表，然后选择 **“行数”**。验证是否出现一组客户。
1. 选择 **“添加新子句”**。
1. 在 **“字段”** 框中，选择 **“stateprovince”**，并在 **“值”** 框中，键入 **“塔斯马尼亚”**。
1. 在工具栏中，选择 **“运行查询”**。验证查询是否返回 106 个行。
1. 在 **“数据资源管理器”** 窗格中，依次展开 **“orderinfo”** 密钥空间、**“orderdetails”** 表，然后选择 **“行数”**。
1. 选择 **“添加新子句”**。
1. 在 **“字段”** 框中，选择 **“customerid”**，并在 **“值”** 框中键入 **“13999”**。
1. 在工具栏中，选择 **“运行查询”**。验证查询是否返回 2 个行。记下第一行的 **“orderid”**（应为 46899）。
1. 在 **“数据资源管理器”** 窗格中，依次展开 **“orderinfo”** 密钥空间、**“orderline”** 表，然后选择 **“行数”**。
1. 选择 **“添加新子句”**。
1. 在 **“字段”** 框中，选择 **“orderid”**，并在 **“值”** 框中键入 **“46899”**。
1. 在工具栏中，选择 **“运行查询”**。验证查询是否返回 1 个行，该行将被订购的产品列出为 **“Road-550-W Yellow, 38”**。

你已通过使用 CQLSH COPY 命令成功地将 Cassandra 数据库迁移到了 Cosmos DB。

### 任务 5：知识总结

1. 切换回在 Cassandra 服务器上运行的 Cassandra Query Shell。该 Shell 仍应连接到你的 Cosmos DB 帐户。如果之前关闭了 Shell，请重新将其打开，如下所示：

    ```bash
    export SSL_VERSION=TLSv1_2
    export SSL_VALIDATE=false

    cqlsh <contact point> <port> -u <username> -p <primary password> --ssl
    ```

1. 在 Cassandra Query Shell 中，运行以下命令以删除密钥空间（和表）：

    ```cqlsh
    drop keyspace customerinfo;
    drop keyspace orderinfo;
    exit;
    ```

## 练习 3：使用 Spark 将数据从 Cassandra 迁移到 Cosmos DB

在本练习中，你将迁移之前使用的相同数据，但是这次你将使用 Azure Databricks 笔记本中的 Spark。

### 任务 1：创建 Spark 群集

1. 在 Azure 门户的左侧窗格中，选择 **“+ 创建资源”**。
1. 在 **“新建”** 页面的 **“搜索市场”** 框，键入 **“Azure Databricks”**，然后按 Enter。
1. 在 **“Azure Databricks”** 页面上选择 **“创建”**。
1. 在 **“Azure Databricks 服务”** 页面上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | 使用现有，assandradbrg |
    | 工作区名称 | CassandraMigration |
    | 位置 | 选择为资源组指定的相同位置 |
    | 定价层 | 标准 |

1. 选择 **“查看 + 创建”**。
1. 在 **“查看 + 创建”** 页面上，选择 **“创建”**，然后等待 Databricks 服务部署完成。
1. 在左侧窗格中，依次选择 **“资源组”**、**“cassandradbrg”**，再选择 **“CassandraMigration”** Databricks 服务。
1. 在 **“CassandraMigration”** 页面上，选择 **“启动工作区”**。
1. 在 **“Azure Databricks”** 页面的 **“常见任务”** 下，选择 **“新建群集”**。
1. 在 **“新建群集”** 页面上，输入以下设置：

    | 属性  | 值  |
    |---|---|
    | 群集名称 | MigrationCluster |
    | 群集模式 | 标准 |
    | 池 | None |
    | Databricks Runtime 版本 | 运行时：5.5 LTS（Scala 2.11、Spark 2.4.3） |
    | Python 版本 | 3 |
    | 启用自动缩放 | 已选择 |
    | 终止时间 | 60 |
    | 辅助角色类型 | 将设置保留为默认设置 |
    | 驱动程序类型 | 与辅助角色相同 |

    > [!备注]
    > 选择运行时版本时，请确保选择不支持 GPU 的版本，否则会收到不兼容错误消息。

1. 选择 **“创建群集”**。
1. 请等待创建群集；群集准备就绪后，**“MigrationCluster”** 的状态将报告为 **“正在运行”**。此过程将花费几分钟时间。

### 任务 2：创建用于迁移数据的笔记本

1. 在左侧窗格中，依次选择 **“群集”**、**“MigrationCluster”**、**“库”** 选项卡，然后选择 **“安装新库”**。
1. 在“**安装库**”对话框中，输入以下设置：

    | 属性  | 值  |
    |---|---|
    | 库源 | Maven |
    | 坐标 | com.datastax.spark：spark-cassandra-connector_2.11：2.4.3 |
    | 存储库 | 留空 |
    | 排除项 | 留空 |

1. 选择 **“安装”**。此库包含用于从 Spark 连接到 Cassandra 的类。
1. 安装连接器库时，选择 **“安装新库”** 以安装其他库。
1. 在 **“安装库”** 对话框中，输入以下设置：

    | 属性  | 值  |
    |---|---|
    | 库源 | Maven |
    | 坐标 | com.microsoft.azure.cosmosdb：azure-cosmos-cassandra-spark-helper：1.2.0 |
    | 存储库 | 留空 |
    | 排除项 | 留空 |

1. 选择 **“安装”**。此库包含用于从 Spark 连接到 Cosmos DB 的类。
1. 在左侧窗格中，选择 **“Azure Databricks”**。
1. 在 **“Azure Databricks”** 页面的 **“常见任务”** 下，选择 **“新建笔记本”**。
1. 在 **“创建笔记本”** 对话框中，输入以下设置：

    | 属性  | 值  |
    |---|---|
    | 名称 | MigrateData |
    | 语言 | Scala |
    | 群集 | MigrationCluster |

1. 选择 **“创建”**。

### 任务 3：连接 Cosmos DB 并创建表

1. 在笔记本的第一个单元格中，输入以下代码：

    ```scala
    // 导入库

    import org.apache.spark.sql.cassandra._
    import org.apache.spark.sql._
    import org.apache.spark._
    import com.datastax.spark.connector._
    import com.datastax.spark.connector.cql.CassandraConnector
    import com.microsoft.azure.cosmosdb.cassandra
    ```

    此代码从 Spark 导入连接到 Cosmos DB 和 Cassandra 所需的类型。

1. 在单元格右侧的工具栏中，选择下拉箭头，然后选择 **“在下方添加单元格”**。
1. 在新单元格中，输入以下代码。使用 Cosmos DB 帐户的值指定“接触点”、“Username”和“主密码”（你在上一个练习中记录了这些值）：

    ```scala
    // 配置 Cosmos DB 的连接参数

    val cosmosDBConf = new SparkConf()
        .set("spark.cassandra.connection.host", "<contact point>")
        .set("spark.cassandra.connection.port", "10350")
        .set("spark.cassandra.connection.ssl.enabled", "true")
        .set("spark.cassandra.auth.username", "<username>")
        .set("spark.cassandra.auth.password", "<primary password>")
        .set("spark.cassandra.connection.factory",
            "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")
        .set("spark.cassandra.output.batch.size.rows", "1")
        .set("spark.cassandra.connection.connections_per_executor_max", "1")
        .set("spark.cassandra.output.concurrent.writes", "1")
        .set("spark.cassandra.concurrent.reads", "1")
        .set("spark.cassandra.output.batch.grouping.buffer.size", "1")
        .set("spark.cassandra.connection.keep_alive_ms", "600000000")
    ```

    此代码设置 Spark 会话参数以连接到 Cosmos DB 帐户

1. 在当前单元格下方添加其他单元格，然后输入以下代码：

    ```scala
    // 创建密钥空间和表

    val cosmosDBConnector = CassandraConnector(cosmosDBConf)

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE KEYSPACE customerinfo WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1}"))
    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE customerinfo.customerdetails (customerid int, firstname text, lastname text, email text, stateprovince text, PRIMARY KEY ((stateprovince), customerid)) WITH cosmosdb_provisioned_throughput=10000"))

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE KEYSPACE orderinfo WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1}"))
    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE orderinfo.orderdetails (orderid int, customerid int, orderdate date, ordervalue decimal, PRIMARY KEY ((customerid), orderdate, orderid)) WITH cosmosdb_provisioned_throughput=10000"))

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE orderinfo.orderline (orderid int, orderline int, productname text, quantity smallint, orderlinecost decimal, PRIMARY KEY ((orderid), productname, orderline)) WITH cosmosdb_provisioned_throughput=10000"))
    ```

    这些语句与表一起重新生成 orderinfo 和 customerinfo 密钥空间。每个表都提供了 10000 RU/秒的吞吐量。

### 任务 4：连接 Cassandra 数据库并检索数据

1. 在笔记本中，添加其他单元格，然后输入以下代码。请将 *“\<ip address\>”* 替换为虚拟机的 IP 地址，并指定之前从 **“bitnami_credentials”** 文件中检索的密码：

    ```scala
    // 为源 Cassandra 数据库配置连接参数

    val cassandraDBConf = new SparkConf()
        .set("spark.cassandra.connection.host", "<ip address>")
        .set("spark.cassandra.connection.port", "9042")
        .set("spark.cassandra.connection.ssl.enabled", "false")
        .set("spark.cassandra.auth.username", "cassandra")
        .set("spark.cassandra.auth.password", "<password>")
        .set("spark.cassandra.connection.connections_per_executor_max", "10")
        .set("spark.cassandra.concurrent.reads", "512")
        .set("spark.cassandra.connection.keep_alive_ms", "600000000")
    ```

1. 添加另一个单元格，然后输入以下代码：

    ```scala
    // 从源数据库检索客户和订单数据

    val cassandraDBConnector = CassandraConnector(cassandraDBConf)
    var cassandraSparkSession = SparkSession
        .builder()
        .config(cassandraDBConf)
        .getOrCreate()

    val customerDataframe = cassandraSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "customerdetails", "keyspace" -> "customerinfo"))
        .load

    println("Read " + customerDataframe.count + " rows")

    val orderDetailsDataframe = cassandraSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderdetails", "keyspace" -> "orderinfo"))
        .load

    println("Read " + orderDetailsDataframe.count + " rows")

    val orderLineDataframe = cassandraSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderline", "keyspace" -> "orderinfo"))
        .load

    println("Read " + orderLineDataframe.count + " rows")
    ```

    这段代码将 Cassandra 数据库中的表中的数据检索到 Spark DataFrame 对象中。代码显示了从每个表中读取的行数。

### 任务 5：将数据插入 Cosmos DB 表并运行笔记本

1. 添加最后一个单元格并输入以下代码：

    ```scala
    // 将客户数据写入 Cosmos DB

    val cosmosDBSparkSession = SparkSession
        .builder()
        .config(cosmosDBConf)
        .getOrCreate()

    // 从 Cosmos DB 连接到现有表
    var customerCopyDataframe = cosmosDBSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "customerdetails", "keyspace" -> "customerinfo"))
        .load

    // 将 Cassandra 数据库中的结果合并到 DataFrame
    customerCopyDataframe = customerCopyDataframe.union(customerDataframe)

    // 将结果写回 Cosmos DB
    customerCopyDataframe.write
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "customerdetails", "keyspace" -> "customerinfo"))
        .mode(org.apache.spark.sql.SaveMode.Append)
        .save()

    // 使用相同的策略将订单数据写入 Cosmos DB
    var orderDetailsCopyDataframe = cosmosDBSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderdetails", "keyspace" -> "orderinfo"))
        .load

    orderDetailsCopyDataframe = orderDetailsCopyDataframe.union(orderDetailsDataframe)

    orderDetailsCopyDataframe.write
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderdetails", "keyspace" -> "orderinfo"))
        .mode(org.apache.spark.sql.SaveMode.Append)
        .save()

    var orderLineCopyDataframe = cosmosDBSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderline", "keyspace" -> "orderinfo"))
        .load

    orderLineCopyDataframe = orderLineCopyDataframe.union(orderLineDataframe)

    orderLineCopyDataframe.write
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderline", "keyspace" -> "orderinfo"))
        .mode(org.apache.spark.sql.SaveMode.Append)
        .save()
    ```

    此代码为 Cosmos DB 数据库中的每个表创建另一个 DataFrame。每个 DataFrame 最初都为空。然后，代码使用 **union** 函数为每个 Cassandra 表添加来自相应 DataFrame 的数据。最后，代码将追加的 DataFrame 写回到 Cosmos DB 表。

    DataFrame API 是 Spark 提供的功能非常强大的抽象，并且是用于快速传输大量数据的高效结构。

1. 在笔记本顶部的工具栏中，选择 **“全部运行”**。  你将看到指示群集正在启动的消息。群集准备就绪后，笔记本将依次在每个单元格中运行代码。你将在每个单元格的下方看到更多消息。读取和写入 DataFrame 的数据传输操作作为 Spark 作业执行。可以展开作业以查看进度。每个单元格中的代码应成功完成，且未显示任何错误消息。

### 任务 6：验证数据迁移是否成功

1. 返回 Azure 门户中的 Cosmos DB 帐户。
1. 选择 **“数据资源管理器”**。
1. 在 **“数据资源管理器”** 窗格中，依次展开 **“customerinfo”** 密钥空间、**“customerdetails”** 表，然后选择 **“行数”**。应显示前 100 行。如果密钥空间没有出现在 **“数据资源管理器”** 窗格中，请选择 **“刷新”** 以更新显示。
1. 依次展开 **“orderinfo”** 密钥空间、**“orderdetails”** 表，然后选择 **“行”**。还应显示此表的前 100 行。
1. 最后，展开“**orderline**”表，然后选择“**行**”。验证是否显示了此表的前 100 行。

你已通过使用 Databricks 笔记本中的 Spark 成功地将 Cassandra 数据库迁移到了 Cosmos DB。

### 任务 7：知识总结

1. 在 Azure 门户的左侧窗格中，选择 **“资源组”**。
1. 在 **“资源组”** 窗口中，选择 **“cassandradbrg”**。
1. 选择 **“删除资源组”**。
1. 在 **“确定要删除‘cassandradbrg’吗”** 页面的 **“输入资源组名称”** 框中，输入 **“cassandradbrg”**，然后选择 **“删除”**。

---
© 2019 Microsoft Corporation. 保留所有权利。

本文档中的文本在 [知识共享署名 3.0
许可](https://creativecommons.org/licenses/by/3.0/legalcode)下提供，并且可能受到附加条款约束。本文档中包含的所有其他内容（包括但不限于商标、徽标、图片等）均
**不**包含在知识共享许可授权范围内。本文档不提供针对任何 Microsoft 产品中任何知识产权的任何合法权利。你可以出于内部参考目的复制和使用本文档。

本文档“按原样”提供。本文档中表达的信息和观点（包括 URL 和其他 Internet 网站参考）如有更改，恕不另行通知。你需要自行承担使用风险。有些示例仅用于说明，而且是虚构的。不能将其视为有意关联或推断。Microsoft 对此处提供的信息不做任何明示或暗示的保证。
