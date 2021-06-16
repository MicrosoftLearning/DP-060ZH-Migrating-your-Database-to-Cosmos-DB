
# 实验室 2：将 MongoDB 工作负载迁移到 Cosmos DB
<!-- TOC -->

- [实验室 2：将 MongoDB 工作负载迁移到 Cosmos DB](#lab-2-migrate-mongodb-workloads-to-cosmos-db)
    - [练习 1：设置](#exercise-1-setup)
        - [任务 1：创建资源组和虚拟网络](#task-1-create-a-resource-group-and-virtual-network)
        - [任务 2：创建一个 MongoDB 数据库服务器](#task-2-create-a-mongodb-database-server)
        - [任务 3：配置 MongoDB 数据库](#task-3-configure-the-mongodb-database)
    - [练习2：填充和查询 MongoDB 数据库](#exercise-2-populate-and-query-the-mongodb-database)
        - [任务 1：生成并运行应用以填充 MongoDB 数据库](#task-1-build-and-run-an-app-to-populate-the-mongodb-database)
        - [任务 2：生成并运行其他应用以查询 MongoDB 数据库](#task-2-build-and-run-another-app-to-query-the-mongodb-database)
    - [练习 3：将 MongoDB 数据库迁移到 Cosmos DB](#exercise-3-migrate-the-mongodb-database-to-cosmos-db)
        - [任务 1：创建 Cosmos 帐户和数据库](#task-1-create-a-cosmos-account-and-database)
        - [任务 2：创建数据库迁移服务](#task-2-create-the-database-migration-service)
        - [任务 3：创建并运行新的迁移项目](#task-3-create-and-run-a-new-migration-project)
        - [任务 4：验证迁移是否成功](#task-4-verify-that-migration-was-successful)
    - [练习 4：重新配置并运行现有应用程序以使用 Cosmos DB](#exercise-4-reconfigure-and-run-existing-applications-to-use-cosmos-db)
    - [练习 5：知识总结](#exercise-5-clean-up)

<!-- /TOC -->
在本实验室中，你将使用现有 MongoDB 数据库并将其迁移到 Cosmos DB。你将使用 Azure 数据库迁移服务。你还将了解如何将使用 MongoDB 数据库的现有应用程序重新配置为连接到 Cosmos DB 数据库。

本实验室以从一系列 IoT 设备中捕获温度数据的示例系统为基础。温度和时间戳一起记录在 MongoDB 数据库中。每个设备都有唯一的 ID。你将运行一个模拟这些设备并将数据存储在数据库中的 MongoDB 应用程序。你还将使用支持用户查询每个设备的统计信息的第二个应用程序。将数据库从 MongoDB 迁移到 Cosmos DB 之后，你将这两个应用程序配置为连接到 Cosmos DB，并验证其是否仍正常运行。

本实验室使用 Azure Cloud Shell 和 Azure 门户运行。

## 练习 1：设置

在第一个练习中，你将创建 MongoDB 数据库，以保存从贵公司制造的温度设备捕获的数据。

### 任务 1：创建资源组和虚拟网络

1. 在 Internet 浏览器中，导航到 https://portal.azure.com 并登录。
1. 在 Azure 门户中，选择 **“资源组”**，然后选择 **“+ 添加”**。
1. 在 **“创建资源组页面”** 上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | mongodbrg |
    | 区域 | 选择离你最近的位置 |

1. 选择 **“查看 + 创建”**，然后选择 **“创建”**。等待资源组创建完成。
1. 在 Azure 门户的侧边栏菜单中，选择 **“+ 创建资源”**。
1. 在 **“新建”** 页面的 **“搜索市场”** 框中，键入 **“虚拟网络”**，然后按 Enter。
1. 在 **“虚拟网络”** 页面上，选择 **“创建”**。
1. 在 **“创建虚拟网络”** 页面上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | mongodbrg |
    | 名称 | databasevnet |
    | 区域 | 选择为资源组指定的相同位置 |

1. 选择 **“下一步: IP 地址”**
1. 在 **“IP 地址”** 页面上的 **“IPv4 地址空间”** 下，选择 **“10.0.0.0/16”** 空间，然后在右侧选择 **“删除”** 按钮。
1. 在 **“IPv4 地址空间”** 列表中，输入 **“10.0.0.0/24”**。
1. 选择 **“+ 添加子网”**。在 **“添加子网”** 窗格中，将 **“子网名称”** 设置为 **“默认值”**，并将 **“子网地址范围”** 设置为 **“10.0.0.0/28”**，然后选择 **“添加”**。
1. 在 **“IP 地址”** 页面上，选择 **“下一页: 安全性”**。
1. 在 **“安全性”** 页面上，确认将 **“DDoS 防护标准”** 设置为 **“禁用”**，并将 **“防火墙”** 设置为 **“禁用”**。选择 **“查看 + 创建”**。
1. 在 **“创建虚拟网络”** 页面上，选择 **“创建”**。等待虚拟网络创建完毕，然后继续。

### 任务 2：创建一个 MongoDB 数据库服务器

1. 在 Azure 门户的侧边栏菜单中，选择 **“+ 创建资源”**。
1. 在 **“搜索市场”** 框中，键入 **“Ubuntu”**，然后按 Enter。
1. 在 **“市场”** 页面上，选择 **“Ubuntu Server 18.04 LTS”**。 
1. 在 **“Ubuntu Server 18.04 LTS”** 页面上，选择 **“创建”**。
1. 在 **“创建虚拟机”** 页面上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 订阅 | *\<your-subscription\>* |
    | 资源组 | mongodbrg |
    | 虚拟机名称 | mongodbserver | 
    | 区域 | 选择为资源组指定的相同位置 |
    | 可用性选项 | 无需基础结构冗余 |
    | 映像 | Ubuntu Server 18.04 LTS - Gen1 |
    | Azure Spot 实例 | 未选中 |
    | 大小 | 标准 A1_v2 |
    | 身份验证类型 | 密码 |
    | 用户名 | azureuser |
    | 密码 | Pa55w.rdPa55w.rd |
    | 确认密码 | Pa55w.rdPa55w.rd |
    | 公共入站端口 | 允许选定的端口 |
    | 选择入站端口 | SSH (22) |

1. 选择 **“下一步: 磁盘 \>”**。
1. 在 **“磁盘”** 页面上，将设置保留为默认值，然后选择 **“下一步: 网络 \>”**。
1. 在 **“网络”** 页面上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 虚拟网络 | databasevnet |
    | 子网 | 默认 (10.0.0.0/28) |
    | 公共 IP | （新）mongodbserver-ip |
    | NIC 网络安全组 | 高级 |
    | 配置网络安全组 | （新）mongodbserver-nsg |
    | 加速网络 | 未选中 |
    | 负载均衡 | 未选中 |

1. 选择 **“查看 + 创建 \>”**。
1. 在验证页面上，选择 **“创建”**。
1. 等待虚拟机部署完毕，然后继续。
1. 在 Azure 门户的侧边栏菜单中，选择 **“所有资源”**。
1. 在 **“所有资源”** 页面上，选择 **“mongodbserver-nsg”**。
1. 在 **“mongodbserver-nsg”** 页面上的 **“设置”** 下，选择 **“入站安全规则”**。
1. 在 **“mongodbserver-nsg - 入站安全规则”** 页面上，选择 **“+ 添加”**。
1. 在 **“添加入站安全规则”** 窗格中，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 源 | 任何 |
    | 源端口范围 | * |
    | 目标 | 任何 |
    | 目标端口范围 | 27017 |
    | 协议 | 任何 |
    | 操作 | 允许 |
    | 优先级 | 1030 |
    | 名称 | Mongodb-port |
    | 描述 | 客户端用于连接到 MongoDB 的端口 |

1. 选择 **“添加”**。

### 任务 3：安装 MongoDB

1. 在 Azure 门户的侧边栏菜单中，选择 **“所有资源”**。
1. 在 **“所有资源”** 页面上，选择 **“mongodbserver-ip”**。
1. 在 **“mongodbserver-ip”** 页面上，记下 **“IP 地址”**。
1. 在 Azure 门户顶部的工具栏中，选择 **“Cloud Shell”**。
1. 如果出现 **“你没有安装存储”** 消息框，请选择 **“创建存储”**。
1. Cloud Shell 启动时，在 Cloud Shell 窗口上方的下拉列表中，选择 **“Bash”**。
1. 在 Cloud Shell 中，输入以下命令以连接 mongodbserver 虚拟机。请将 *“\<ip address\>”* 替换为 **“mongodbserver-ip”** IP 地址的值：

    ```bash
    ssh azureuser@<ip address>
    ```

1. 在提示符下，键入 **“是”** 以继续连接。
1. 输入密码 **“Pa55w.rdPa55w.rd”**。
1. 输入以下命令，以导入 MongoDB 公共 GPG 密钥。该命令应当返回 **“确定”**：

    ```bash
    wget -qO - https://www.mongodb.org/static/pgp/server-4.0.asc | sudo apt-key add -
    ```

1. 要创建包列表，请输入以下命令：

    ```bash
    echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
    ```

    该命令应返回的文本类似于 `deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.0 multiverse`。这表示已成功创建包列表。

1. 要重新加载包数据库，请输入以下命令：

    ```bash
    sudo apt-get update
    ```

1. 要安装 MongoDB，请输入以下命令：

    ```bash
    sudo apt-get install -y mongodb-org
    ```

    安装过程应会继续出现有关安装、准备和解压缩包的消息。安装可能需要几分钟的时间才能完成。

### 任务 4：配置 MongoDB 数据库

默认情况下，Mongo DB 实例配置为无需身份验证即可运行。在此任务中，你将 MongoDB 配置为绑定到本地网络接口，以便它可以接受来自其他计算机的连接。你还将启用身份验证并创建必要的用户帐户以执行迁移。最后，你将添加一个帐户，测试应用程序可以使用该帐户查询数据库。

1. 要打开 MongoDB 配置文件，请运行以下命令：

    ```bash
    sudo nano /etc/mongod.conf
    ```

1. 在文件中，找到 **“bindIp”** 设置，并将其设置为 **“0.0.0.0”**。
1. 添加以下设置。如果已有 `security` 部分，请用此代码替换它。否则，请在两个新行中添加该代码：

    ```bash
    security:
        authorization: 'enabled'
    ```

1. 要保存配置文件，请按 <kbd>Esc</kbd>，然后按 <kbd>CTRL + X</kbd>。按 <kbd>y</kbd>，然后按 <kbd>Enter</kbd> 保存修改后的缓冲区。
1. 要重新启动 MongoDB 服务并应用所做的更改，请输入以下命令：

    ```bash
    sudo service mongod restart
    ```

1. 要连接到 MongoDB 服务，请输入以下命令：

    ```bash
    mongo
    ```

1. 在 **“>”** 提示符下，要切换到 **admin** 数据库，请运行以下命令：

    ```bash
    use admin;
    ```

1. 要创建名为 **administrator** 的新用户，请运行以下命令。可以在一行或多行中输入命令以提高可读性。当 `mongo` 程序到达分号时，将执行命令：

    ```bash
    db.createUser(
        {
            user: "administrator",
            pwd: "Pa55w.rd",
            roles: [
                { role: "userAdminAnyDatabase", db: "admin" },
                { role: "clusterMonitor", db:"admin" },
                "readWriteAnyDatabase"
            ]
        }
    );
    ```

1. 要退出 `mongo` 程序，请输入以下命令：

    ```bash
    exit;
    ```

1. 要使用新管理员的帐户连接到 MongoDB，请运行以下命令：

    ```bash
    mongo -u "administrator" -p "Pa55w.rd"
    ```

1. 要切换到 **DeviceData** 数据库，请执行以下命令：

    ```bash
    use DeviceData;    
    ```

1. 要创建名为 **deviceadmin** 的用户（应用将使用该用户名连接到数据库），请运行以下命令：

    ```bash
    db.createUser(
        {
            user: "deviceadmin",
            pwd: "Pa55w.rd",
            roles: [ { role: "readWrite", db: "DeviceData" } ]
        }
    );
    ```

1. 要退出 `mongo` 程序，请输入以下命令：

    ```bash
    exit;
    ```

1. 运行以下命令，重启 mongodb 服务。验证服务是否重新启动，且没有出现任何错误消息：

    ```bash
    sudo service mongod restart
    ```

1. 运行以下命令，以验证你现在是否能够以 deviceadmin 用户身份登录 mongodb：

    ```bash
    mongo -u "deviceadmin" -p "Pa55w.rd" --authenticationDatabase DeviceData
    ```

1. 在 **“>”** 提示符下，运行以下命令以退出 mongo shell：

    ```bash
    exit;
    ```

1. 在 bash 提示符下，运行以下命令以断开与 MongoDB 服务器的连接并返回到 Cloud Shell：

    ```bash
    exit
    ```

## 练习2：填充和查询 MongoDB 数据库

你现在已经创建了 MongoDB 服务器和数据库。下一步是演示可以填充和查询此数据库中的数据的示例应用程序。

### 任务 1：生成并运行应用以填充 MongoDB 数据库

1. 在 Azure Cloud Shell 中，运行以下命令以下载此研讨会的示例代码：

    ```bash
    git clone https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB migration-workshop-apps
    ```

1. 移至 **“migration-workshop-apps/MongoDeviceDataCapture/MongoDeviceCapture”** 文件夹：

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/MongoDeviceDataCapture
    ```

1. 使用“代码”编辑器来检查 **“TemperatureDevice.cs”** 文件：

    ```bash
    code TemperatureDevice.cs
    ```

    该文件中的代码包含名为 **TemperatureDevice** 的类，该类可模拟捕获数据并将其保存在 MongoDB 数据库中的温度设备。该类使用适用于 .NET Framework 的 MongoDB 库。**“TemperatureDevice”** 构造函数使用存储在应用程序配置文件中的设置连接到数据库。**“RecordTemperatures”** 方法生成读数并将其写入数据库。

1. 要关闭代码编辑器，请按 <kbd>CTRL + Q</kbd>，然后打开 **“ThermometerReading.cs”** 文件：

   ```bash
   code ThermometerReading.cs
   ```

    此文件显示了应用程序存储在数据库中的文档的结构。每个文档都包含以下字段：

    - 对象 ID。它是 MongoDB 生成的“_id”字段，用于唯一标识每个文档。
    - 设备 ID。每个设备都有一个前缀为“Device”的数字。
    - 设备记录的温度。
    - 记录温度的日期和时间。
  
1. 要关闭代码编辑器，请按 <kbd>CTRL + Q</kbd>，然后打开 **“App.config”** 文件：

    ```bash
    code App.config
    ```

    此文件包含用于连接到 MongoDB 数据库的设置。将 **“地址”** 项的值设置为之前记录的 MongoDB 服务器的 IP 地址。

1. 要保存文件，请按 <kbd>CTRL + S</kbd>，然后按 <kbd>CTRL + Q</kbd> 关闭代码编辑器
1. 要打开项目文件，请输入以下命令：

    ```bash
    code MongoDeviceDataCapture.csproj
    ```

1. 检查 `<TargetFramework>` 标记是否包含值 `netcoreapp3.1`。如有必要，请替换此值。
1. 要保存文件，请按 <kbd>CTRL + S</kbd>，然后按 <kbd>CTRL + Q</kbd> 关闭代码编辑器
1. 运行以下命令以重新生成应用程序：

    ```bash
    dotnet build
    ```

1. 运行应用程序：

    ```bash
    dotnet run
    ```

    应用程序模拟了 100 个同时运行的设备。让应用程序运行几分钟，然后按 Enter 停止运行。

### 任务 2：生成并运行另一个应用以查询 MongoDB 数据库

1. 移至 **“migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery”** 文件夹：

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

    此文件夹包含另一个应用程序，可使用该应用程序分析由每台设备捕获的数据。

1. 使用“代码”编辑器来检查 **“Program.cs”** 文件：

    ```bash
    code Program.cs
    ```

    应用程序连接到数据库（使用文件底部的 **ConnectToDatabase** 方法），然后提示用户输入设备编号。该应用程序使用适用于 .NET Framework 的 MongoDB 库来创建和运行聚合管道，该管道可为指定设备计算以下统计信息：

    - 记录的读数数目。
    - 记录的平均温度。
    - 最小读数。
    - 最高读数。
    - 最新读数。

1. 要关闭代码编辑器，请按 <kbd>CTRL + Q</kbd>，然后打开 **“App.config”** 文件：

    ```bash
    code App.config
    ```

    与之前一样，将 **“地址”** 项的值设置为之前记录的 MongoDB 服务器的 IP 地址。

1. 要保存文件，请按 <kbd>CTRL + S</kbd>，然后按 <kbd>CTRL + Q</kbd> 关闭代码编辑器
1. 要打开项目文件，请输入以下命令：

    ```bash
    code DeviceDataQuery.csproj
    ```

1. 检查 `<TargetFramework>` 标记是否包含值 `netcoreapp3.1`。如有必要，请替换此值。
1. 要保存文件，请按 <kbd>CTRL + S</kbd>，然后按 <kbd>CTRL + Q</kbd> 关闭代码编辑器

1. 生成并运行应用程序：

    ```bash
    dotnet build
    dotnet run
    ```

1. 在 **“输入设备编号”** 提示符下，请输入一个介于 0 和 99 之间的值。应用程序将查询数据库、计算统计信息并显示结果。按 <kbd>Q + Enter</kbd> 退出应用程序。

## 练习 3：将 MongoDB 数据库迁移到 Cosmos DB

下一步是获取 MongoDB 数据库并将其转移到 Cosmos DB。

### 任务 1：创建 Cosmos 帐户和数据库

1. 返回 Azure 门户。
1. 在侧边栏菜单中，选择 **“+ 创建资源”**。
1. 在 **“新建”** 页面上的 **“在市场中搜索”** 框中，键入 **“Azure Cosmos DB”**，最后按 Enter。
1. 在 **“Azure Cosmos DB”** 页面上，选择 **“创建”**。
1. 在 **“创建 Azure Cosmos DB 帐户”** 页面上，输入以下设置：

    | 属性  | 值  |
    |---|---|
    | 订阅 | 选择你的订阅 |
    | 资源组 | mongodbrg |
    | 帐户名 | mongodb*nnn*，其中 *“nnn”* 表示你选择的随机编号 |
    | API | 适用于 MongoDB API 的 Azure Cosmos DB |
    | Notebooks | 关 |
    | 位置 | 指定用于 MongoDB 服务器和虚拟网络的相同位置 |
    | 容量模式 | 预配的吞吐量 |
    | 应用免费层折扣 | 应用 |
    | 帐户类型 | 非生产 |
    | 版本 | 3.6 |
    | 异地冗余 | 禁用 |
    | 多区域写入 | 禁用 |
    | 可用性区域 | 禁用 |

1. 选择 **“查看 + 创建”**
1. 在验证页面上，选择 **“创建”**，然后等待 Cosmos DB 帐户完成部署。
1. 在 Azure 门户的侧边栏菜单中，选择 **“所有资源”**，然后选择新建的 Cosmos DB 帐户 (mongodbnnn)。
1. 在 **“mongodbnnn”** 页面上，选择 **“数据资源管理器”**。
1. 在 **“数据资源管理器”** 窗格中，选择 **“新建集合”**。
1. 在 **“添加集合”** 窗格中，指定以下设置：

    | 属性  | 值  |
    |---|---|
    | 数据库 ID | 选择 **“新建”**，然后键入 **“DeviceData”** |
    | 预配数据库吞吐量 | 已选中 |
    | 吞吐量 | 1000 |
    | 集合 ID | 温度 |
    | 存储容量 | 无限制 |
    | 分片键 | deviceID |
    | 我的分片键超过 100 个字节 | unchecked |
    | 在所有字段上创建通配符索引 | unchecked |
    | 分析存储 | 关 |

1. 选择 **“确定”**。

### 任务 2：创建数据库迁移服务

1. 在 Azure 门户的侧边栏菜单中，选择 **“所有服务”**。
1. 在 **“所有服务”** 搜索框中，键入 **“Subscriptions”**，然后按 Enter。
1. 在 **“订阅”** 页面上，选择你的订阅。
1. 在你的订阅页面上，选择“**设置**”下方的“**资源提供程序**”。
1. 在“**按名称筛选**”方框中，键入“**DataMigration**”，然后选择“**Microsoft.DataMigration**”。
1. 如果 **“状态”** 不是 **“已注册”**，请选择 **“注册”**，然后等待 **“状态”** 更改为 **“已注册”**。可能需要选择 **“刷新”** 以查看状态更改。
1. 在 Azure 门户的侧边栏菜单中，选择 **“+ 创建资源”**。
1. 在 **“新建”** 页面的“搜索市场”框中，键入 **“Azure 数据库迁移服务”**，然后按 Enter。
1. 在 **“Azure 数据库迁移服务”** 页面上，选择 **“创建”**。
1. 在 **“创建迁移服务”** 页面上，输入以下设置：

    | 属性  | 值  |
    |---|---|
    | 订阅 | 选择你的订阅 |
    | 资源组 | mongodbrg |
    | 服务名称 | MongoDBMigration |
    | 位置 | 选择你以前使用的同一位置 |
    | 服务模式 | Azure |
    | 定价层 | 标准：1 个 vCore |

1. 选择 **“下一步: 网络”**。
1. 在 **“网络”** 页面上，选中 **“databasevnet/default”** 复选框，然后选择 **“查看 + 创建”**
1. 选择 **“创建”**，等待服务部署完毕，然后再继续。此操作将需要几分钟的时间。

### 任务 3：创建并运行新的迁移项目

1. 在 Azure 门户的侧边栏菜单中，选择 **“资源组”**。
1. 在 **“资源组”** 窗口中，选择 **“mongodbrg”**。
1. 在 **“mongodbrg”** 窗口中，选择 **“MongoDBMigration”**。
1. 在 **“MongoDBMigration”** 页面上，选择 **“+ 新建迁移项目”**。
1. 在 **“新建迁移项目”** 页面上，输入以下设置：

    | 属性  | 值  |
    |---|---|
    | 项目名称 | MigrateTemperatureData |
    | 源服务器类型 | MongoDB |
    | 目标服务器类型 | Cosmos DB (MongoDB API) |
    | 选择活动类型 | 离线数据迁移 |

1. 选择 **“创建并运行活动”**。
1. **“迁移向导”** 启动后，在 **“源详细信息”** 页上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 模式 | 标准模式 |
    | 源服务器名称 | 指定之前记录的 **“mongodbserver-ip”** IP 地址的值 |
    | 服务器端口 | 27017 |
    | 用户名 | administrator |
    | 密码 | 中的机器人 Pa55w.rd |
    | 需要 SSL | unchecked |

1. 选择 **“下一步: 选择目标”**。
1. 在 **“选择目标”** 页面上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 模式 | 选择 Cosmos DB 目标 |
    | 订阅 | 选择你的订阅 |
    | 选择 Cosmos DB 名称 | mongodb*nnn* |
    | 连接字符串 | 接受为 Cosmos DB 帐户生成的连接字符串 |

1. 选择 **“下一步: 数据库设置”**。
1. 在 **“数据库设置”** 页面上，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 源数据库 | DeviceData |
    | 目标数据库 | DeviceData |
    | 吞吐量（RU/秒） | 1000 |
    | 清理集合 | 未选中 |

1. 选择 **“下一步: 集合设置”**。
1. 在 **“集合设置”** 页面上，选择 DeviceData 数据库旁边的下拉箭头，输入以下详细信息：

    | 属性  | 值  |
    |---|---|
    | 名称 | 温度 |
    | 目标集合 | 温度 |
    | 吞吐量（RU/秒） | 1000 |
    | 分片键 | deviceID |
    | 唯一 | 留空 |

1. 选择 **“下一步: 迁移摘要”**
1. 在 **“迁移摘要”** 页上的 **“活动名称”** 字段中，输入 **“mongodb-migration”**，然后选择 **“开始迁移”**。
1. 在 **“mongodb-migration”** 页面上，每隔 30 秒选择一次 **“刷新”**，直到迁移完成。注意已处理的文档数。

### 任务 4：验证迁移是否成功

1. 在 Azure 门户的侧边栏菜单中，选择 **“所有资源”**。
1. 在 **“所有资源”** 页面上，选择 **“mongodbnnn”**。
1. 在 **“mongodbnnn”** 页面上，选择 **“数据资源管理器”**。
1. 在 **“数据资源管理器”** 窗格中，依次展开 DeviceData 数据库和 **“温度”** 集合，然后选择 **“文档”**。
1. 在 **“文档”** 窗格中，滚动浏览文档列表。你应看到每个文档的文档 ID (_id) 和共享密钥 (/deviceID)。
1. 选择任意文档。应会看到显示的文档的详细信息。典型的文档如下所示：

    ```JSON
    {
	    "_id" : ObjectId("5ce8104bf56e8a04a2d0929a"),
	    "deviceID" : "Device 83",
	    "temperature" : 19.65268837271849,
	    "time" : 636943091952553500
    }
    ```

1. 在 **“文档资源管理器”** 窗格中的工具栏中，选择 **“新建 Shell”**。
1. 在 **“Shell 1”** 窗格中，在 **“\>”** 提示符下，输入以下命令，然后按 Enter：

    ```mongosh
    db.Temperatures.count()
    ```

    此命令显示“温度”集合中的文档数量。文档数量应与“迁移向导”报告的数量一致。

1. 输入以下命令，然后按 Enter：

    ```mongosh
    db.Temperatures.find({deviceID: "Device 99"})
    ```

    此命令提取并显示设备 99 的相关文档。

## 练习 4：重新配置并运行现有应用程序以使用 Cosmos DB

最后一步是重新配置现有 MongoDB 应用程序以连接到 Cosmos DB，并验证它们是否照常\运行。此过程要求修改应用程序连接数据库的方式，但是应用程序的逻辑应保持不变。

1. 在 **“mongodb*nnn”** 窗格中的 **“设置”** 下，选择 **“连接字符串”**。
1. 在 **“mongodb*nnn* 连接字符串”** 页面上，记下以下设置：

    - 主机
    - 用户名
    - 主密码
  
1. 返回 Cloud Shell 窗口（如果会话超时，请重新连接），然后移至 **“migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery”** 文件夹：

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

1. 在代码编辑器中打开 App.config 文件：

    ```bash
    code App.config
    ```

1. 在文件的 **“MongoDB 设置”** 部分中，注释掉现有设置。
1. 取消注释 **“Cosmos DB Mongo API 设置”** 部分中的设置，并按如下所示设置这些设置的值：

    | 设置  | 值  |
    |---|---|
    | 地址 | **“mongodb*nnn* 连接字符串”** 页面中的主机 |
    | 用户名 | **“mongodb*nnn* 连接字符串”** 页面中的 **“Username”** |
    | 密码 | **“mongodb*nnn* 连接字符串”** 页面中的主密码 |

    完成的文件应如下所示：

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="Database" value="DeviceData" />
            <add key="Collection" value="Temperatures" />

            <!-- Settings for MongoDB -->
            <!--add key="Address" value="nn.nn.nn.nn" />
            <add key="Port" value="27017" />
            <add key="Username" value="deviceadmin" />
            <add key="Password" value="Pa55w.rd" /-->
            <!-- End of settings for MongoDB -->

            <!-- Settings for CosmosDB Mongo API -->
            <add key="Address" value="mongodbnnn.documents.azure.com"/>
            <add key="Port" value="10255"/>
            <add key="Username" value="mongodbnnn"/>
            <add key="Password" value="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=="/>
            <!-- End of settings for CosmosDB Mongo API -->
        </appSettings>
    </configuration>
    ```

1. 保存文件，然后关闭代码编辑器。

1. 使用代码编辑器打开 Program.cs 文件：

    ```bash
    code Program.cs
    ```

1. 向下滚动到 **ConnectToDatabase** 方法。
1. 注释掉设置用于连接 MongoDB 的凭据的行，并取消注释指定用于连接 Cosmos DB 的凭据的语句。代码应如下所示：

    ```C#
    // 连接 MongoDB 数据库
    MongoClient client = new MongoClient(new MongoClientSettings
    {
        Server = new MongoServerAddress(address, port),
        ServerSelectionTimeout = TimeSpan.FromSeconds(10),

        //
        // MongoDB 的凭据设置
        //

        // 凭据 = MongoCredential.CreateCredential(database, azureLogin.UserName, azureLogin.SecurePassword),

        //
        // CosmosDB Mongo API 的凭据设置
        //

        UseTls = true,
        Credential = new MongoCredential("SCRAM-SHA-1", new MongoInternalIdentity(database, azureLogin.UserName), new PasswordEvidence(azureLogin.SecurePassword))

        // Mongo API 设置结束 
    });

    ```

    这些更改是必需的，因为原始的 MongoDB 数据库未使用 TLS 连接。Cosmos DB 始终使用 TLS。

1. 保存文件，然后关闭代码编辑器。
1. 重新生成并运行应用程序：

    ```bash
    dotnet build
    dotnet run
    ```

1. 在 **“输入设备编号”** 提示符下，请输入一个介于 0 和 99 之间的设备编号。应用程序应照常运行，只是这次使用的是 Cosmos DB 数据库中保存的数据。
1. 使用其他设备编号测试应用程序。输入 **“Q”** 完成操作。

你已成功将 MongoDB 数据库迁移到 Cosmos DB，并将现有 MongoDB 应用程序重新配置为连接到 Cosmos DB 数据库。

## 练习 5：知识总结

1. 返回 Azure 门户。
1. 在侧边栏菜单中，选择 **“资源组”**。
1. 在 **“资源组”** 窗口中，选择 **“mongodbrg”**。
1. 选择 **“删除资源组”**。
1. 在 **“确定要删除‘mongodbrg’吗”** 页面的 **“输入资源组名称”** 框中，输入 **“mongodbrg”**，然后选择 **“删除”**。

---
© 2020 Microsoft Corporation. 保留所有权利。

本文档中的文本在[知识共享署名 3.0 许可](https://creativecommons.org/licenses/by/3.0/legalcode)下提供，并且可能受到附加条款约束。本文档中包含的所有其他内容（包括但不限于商标、徽标、图片等）均 **不** 包含在知识共享许可授权范围内。本文档不提供针对任何 Microsoft 产品中任何知识产权的任何合法权利。你可以出于内部参考目的复制和使用本文档。

本文档“按原样”提供。本文档中表达的信息和观点（包括 URL 和其他 Internet 网站参考）如有更改，恕不另行通知。你需要自行承担使用风险。有些示例仅用于说明，而且是虚构的。不能将其视为有意关联或推断。Microsoft 对此处提供的信息不做任何明示或暗示的保证。
