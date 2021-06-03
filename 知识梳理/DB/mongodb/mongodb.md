# 错误处理

## 关闭时数据损坏，无法打开

```shell
rm -f /mongodb/single/data/db/*.lock
/usr/local/mongodb/bin/mongod --repair --dbpath=/mongodb/single/data/db
```

# 副本集

副本集：维护相同 mongod 数据集的服务，副本集提供冗余和高可用

1. 在不同数据库服务器上提供多个数据副本，提供一定的容错能力，防止单个服务器数据丢失
2. 副本集包含**数据承载节点**和**仲裁节点**。

  - 承载数据节点：在承载数据的节点中，存在一个主节点和多个副本节点，主节点接收所有操作，副本节点不能进行操作（但可以配置为只读）。
  - 仲裁节点：选举节点，当主节点挂了，仲裁节点从副本集内部选举一个新的主节点

**副本集和主从集群区别**

1. 副本集没有固定的主节点，主节点是由副本节点选举出来的
2. 整个副本集只存在一个主节点和多个副本节点，当前主节点挂了，从剩余的副本节点继续选举

## 副本集角色

1. Primary: 主节点，接收所有读写操作
2. Secondary: 副本节点，通过复制主节点的操作以维护相同的数据，不可操作，但可以配置为只读
3. Arbiter: 仲裁者，不保留任何数据，只具有投票选举。**也可以把仲裁者维护成副本节点**，这样一个节点就同时存在两个角色，可以理解为一个副本节点，同时具有仲裁功能

![image.png](https://cdn.nlark.com/yuque/0/2020/png/262797/1609316657913-b650c84b-8bff-4156-84a1-a4581b22658a.png#align=left&display=inline&height=312&margin=%5Bobject%20Object%5D&name=image.png&originHeight=624&originWidth=1568&size=101992&status=done&style=none&width=784)

## 选举

**选举条件**：

1. 主节点故障
2. 主节点网络不可达，默认心跳为 10s
3. 人工干预 rs.stepDown(600);

**选举原则**：

1. 票数最高，并且必须要获得大部分成员的投票。**什么叫大部分成员？**至少大于一半。例如三个投票成员，则大部分至少是 2。当复制集内存活的成员不足大多数时，则整个副本集无法选举出主节点，则服务降级，只提供读取服务
2. 如果票数相同，则数据新的节点获胜，数据的新旧是通过操作日志 oplog 来对比的。

数据比较时，节点优先级非常重要，优先级可以理解为票数，(0-1000)，如果选举节点优先级为 1000，则它一次投票就为 1000。主节点和其他副节点优先级默认都是 1，选举节点必须是 0，（即不具备选举权，但可以投票）

**选举时，请千万注意大多数，如果只存在一个节点的情况下，尽管他获取的票数肯定是最高，但他不可能是大多数，所以会服务降级**！！！

## 副本集操作

### 初始化

当连接上数据库时，需要命令是无法使用的，例如 show dbs，需要初始化副本集才可以进行操作

```shell
rs.initiate(configuration); # configuration 不传则为默认配置
```

[replica-set-configuration-document](https://docs.mongodb.com/manual/reference/replica-configuration/#replica-set-configuration-document)

### 查看

```shell
rs.conf(); # configuration：可选，如果没有配置，则使用默认主节点配置
rs.config();

cfg 部分属性
  _id : 副本集配置的存储主键值，默认为副本集名称

  members: 副本节点
      _id: 主键id
      host: 地址
      arbiterOnly: 是否是仲裁节点
      priority: 权重

  settings : 副本集配置参数

cfg=rs.conf(); # cfg 用 cfg 可以赋值啥的，改变配置
rs.reconfig(cfg) # 重新加载配置


rs.status() # 查看副本集状态
```

### 配置

```shell
rs.add(host, arbiterOnly); # 添加其他副本节点，arbiterOnly是否为仲裁节点
rs.addArb(host); # 添加仲裁节点
rs.remove(); # 删除

rs.slaveOk(); # 配置副本节点只读为，db.getMongo().setSlaveOk() 简化命令
rs.slaveOk(true); # 配置副本节点只读
rs.slaveOk(false); # 取消副节点的读
```

## 副本集示例

3 个节点（一主一副一仲裁）

### 副本成员（PRIMARY）

```yaml
systemLog:
  # MongoDB 发送所有日志输出的目标指定为文件
  destination: file

  # MongoDB 应向其发送所有诊断日志记录信息的日志文件的路径
  path: '~/mongodb/cluster1/log/mongod.log'

  #当 mongo 或 mongo 实例重新启动时，mongo 或 mongo 会将新条目附加到现有日志文件的末尾。
  logAppend: true

storage:
  # MongoDB 实例存储其数据的目录。storage.dbPath 设置仅适用于 mongod。
  dbPath: '~/mongodb/cluster1/data/db'

  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true

processManagement:
  #启用在后台运行 MongoDB 进程的守护进程模式。
  fork: true

  #指定用于保存MongoDB的进程ID的文件位置，其中MongoDB将写入其PID
  pidFilePath: '~/mongodb/cluster1/log/mongod.pid'

net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  bindIp: localhost #服务实例绑定的IP
  port: 27017 #绑定的端口
replication:
  replSetName: test_cluster #副本集的名称
```

### 副本成员（SECONDARY）

```yaml
systemLog:
  # MongoDB 发送所有日志输出的目标指定为文件
  destination: file

  # MongoDB 应向其发送所有诊断日志记录信息的日志文件的路径
  path: '~/mongodb/cluster2/log/mongod.log'

  #当 mongo 或 mongo 实例重新启动时，mongo 或 mongo 会将新条目附加到现有日志文件的末尾。
  logAppend: true

storage:
  # MongoDB 实例存储其数据的目录。storage.dbPath 设置仅适用于 mongod。
  dbPath: '~/mongodb/cluster2/data/db'

  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true

processManagement:
  #启用在后台运行 MongoDB 进程的守护进程模式。
  fork: true

  #指定用于保存MongoDB的进程ID的文件位置，其中MongoDB将写入其PID
  pidFilePath: '~/mongodb/cluster2/log/mongod.pid'

net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  bindIp: localhost #服务实例绑定的IP
  port: 27018 #绑定的端口
replication:
  replSetName: test_cluster #副本集的名称
```

### 仲裁者

```yaml
systemLog:
  # MongoDB 发送所有日志输出的目标指定为文件
  destination: file

  # MongoDB 应向其发送所有诊断日志记录信息的日志文件的路径
  path: '~/mongodb/cluster3/log/mongod.log'

  #当 mongo 或 mongo 实例重新启动时，mongo 或 mongo 会将新条目附加到现有日志文件的末尾。
  logAppend: true

storage:
  # MongoDB 实例存储其数据的目录。storage.dbPath 设置仅适用于 mongod。
  dbPath: '~/mongodb/cluster3/data/db'

  journal:
    #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
    enabled: true

processManagement:
  #启用在后台运行 MongoDB 进程的守护进程模式。
  fork: true

  #指定用于保存MongoDB的进程ID的文件位置，其中MongoDB将写入其PID
  pidFilePath: '~/mongodb/cluster3/log/mongod.pid'

net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  bindIp: localhost #服务实例绑定的IP
  port: 27019 #绑定的端口
replication:
  replSetName: test_cluster #副本集的名称
```

# 分片

分片（sharding）是一种**跨多台机器分布数据**的方法， MongoDB 使用分片来支持具有非常大的数据集和高吞吐量操作的部署，其实就是 Mongodb 将数据拆分，放到不同的机器上，以提供更高的负载

通常有两种扩展方式

1. 垂直扩展：扩展单个服务器容量，简单来说就是增加配置
2. 水平扩展：划分数据及并配置更多的服务器，以添加服务器来代替添加单个服务器配置，且具有更好的扩容能力，增加服务器就是了

## 分片集群成员

1. 分片（存储，使用副本集存储）：可以部署多个副本集
2. Config Server：设置服务器存储集群的元数据以及配置，从 3.4 开始配置服务器也必须是副本集
3. mongos Router：充当查询路由，在客户端和分片集群之间提供接口

角色：

1. shardsvr（碎片）：单个 Mongod 或副本集，用于存储分片集群总数据的某些部分，在生产中，所有分片都应该是副本集
2. configsvr（配置服务器）：一个 mongod 实例，它存储与分片群集关联的所有元数据

![image.png](https://cdn.nlark.com/yuque/0/2020/png/262797/1609316342173-b4881f4e-8237-40a3-aad0-c2ff3e54f4b9.png#align=left&display=inline&height=550&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1100&originWidth=1570&size=229208&status=done&style=none&width=785) 设置 sharding.clusterRole 需要 mongod 实例运行复制。 要将实例部署为副本集成员，请使用 replication.replSetName 设置并指定副本集的名称

```shell
mongos -f mongos.conf # 启动分片路由，请注意，副本集是 mongos，不是 mongod

sh.addShard("IP:Port") # 添加分片
sh.status() # 查看分片集群状态


use admin
db.runCommand( { removeShard: "myshardrs02" } ) # 移除分片，移除时会自动转移数据，这需要时间，转移完成后，再执行命令即可删除

sh.enableSharding("库名") # 开启分片功能
sh.shardCollection(namespace, key, unique)

  namespace: string  # 库名.表名
  key: document # 用作分片键的索引规范文档。shard键决定MongoDB如何在 shard之间分发文档。除非集合为空，否则索引必须在shard collection命令之前存在
  unique boolean # 当值为true情况下，片键字段上会限制为确保是唯一索引。哈希策略片键不支持唯一索引。默认是false

db.printShardingStatus() # 显示集群信息
sh.isBalancerRunning() # 集群是否工作
sh.getBalancerState() # 集群均衡器工作状态
```

key 是决定分片策略的关键，具有如下特点

1. 一个集合只能指定一个片键，否则报错
2. 一旦对一个集合分片，分片键和分片值就不可改变。 如：不能给集合选择不同的分片键、不能更新分片键的值
3. 根据 age 索引进行分配数据

**分片规则一：哈希策略**

对于 基于哈希的分片 ,MongoDB 计算一个字段的哈希值,并用这个哈希值来创建数据块，在使用基于哈希分片的系统中,拥有"相近"片键的文档 很可能不会 存储在同一个数据块中，因此数据的分离性更好一些

**分片规则二：范围策略**

对于 基于范围的分片 ,MongoDB 按照片键的范围把数据分成不同部分.假设有一个数字的片键:想象一个从负无穷到正无穷的直线,每一个片键的值都在直线上画了一个点.MongoDB 把这条直线划分为更短的不重叠的片段,并称之为 数据块 ,每个数据块包含了片键在一定范围内的数据.

1. 范围策略提供更高效的范围搜索，给定一个范围，分发路由很块可以确定数据的存储块；哈希路由对于范围查询较弱
2. 范围查询会导致一个问题，数据在分片的分布不平衡，可能造成在一定时间内，请求都落在一个分片内，导致服务器压力巨大；哈希策略随机性是数据均匀的分布在各个分片中，但正因为随机性，无法快速的响应范围查询，因为你得请求所有分片才能确定范围

如无特殊情况，一般推荐使用 Hash Sharding，一般都可以选择_id 作为 Key

## 分片示例

### 部署分片副本集

和副本集保持一致，但需要增加新的属性

```yaml
sharding:
  #分片角色
  clusterRole: shardsvr
```

### 部署配置节点

和副本集保持一致，但需要增加新的属性

```yaml
sharding:
  clusterRole: configsvr
```

### 部署路由

```yaml
systemLog:
  #MongoDB发送所有日志输出的目标指定为文件
  destination: file
  #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
  path: '/mongodb/sharded_cluster/mymongos_27017/log/mongod.log'
  #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
  logAppend: true
processManagement:
  #启用在后台运行mongos或mongod进程的守护进程模式。
  fork: true
  #指定用于保存mongos或mongod进程的进程ID的文件位置，其中mongos或mongod将写入其PID
  pidFilePath: /mongodb/sharded_cluster/mymongos_27017/log/mongod.pid"
net:
  #服务实例绑定所有IP，有副作用，副本集初始化的时候，节点名字会自动设置为本地域名，而不是ip
  #bindIpAll: true
  #服务实例绑定的IP
  bindIp: localhost
  #bindIp
  #绑定的端口
  port: 27017
sharding:
  #指定配置节点副本集
  configDB: myconfigrs/180.76.159.126:27019,180.76.159.126:27119,180.76.159.126:27219
```

# 权限

**增加安全举措**

1. 改变端口，不用 27017
2. 内网使用
3. 开启安全认证，启动时添加 `--auth` 或配置项为 `auth: true`

  ## 角色

角色                  | 权限描述
------------------- | -----------------------------------------------------------------
read                | 可以读取指定数据库中任何数据
readWrite           | 可以读写指定数据库中任何数据，包括创建、重命名、删除集合。
readnyDatabase      | 可以读取所有数据库中任何数据(除了数据库 config 和 local 之外)。
readWriteAnyatabase | 可以读写所有数据库中任何数据(除了数据库 config 和 local 之外)。
userAdminAnyatabase | 可以在指定数据库创建和修改用户(除了数据库 config 和 local 之外)。
dbAdminAnyDatabase  | 可以读取任何数据库以及对数据库进行理、修改、压缩、获取统 计信息、执行检查等操作(除了数据库 config 和 local 之外)
dbAdmin             | 可以读取指定据库以及对数据库进行清理、修改、压缩、获取统 计信息、执行检查等操作。
userAdmin           | 可以在指定数据库创建和修改用户。
clusterAdmin        | 可以对整个集群或数据库统进行管理操作。
backup              | 备份 MongoDB 数据最小的权限。
restore             | 从备文件中还原恢复 MongoDB 数据(除了 system.profile 集合)的权限。
root                | 超级账号，超级权限

## 查询角色权限

```shell
db.runCommand({ rolesInfo: 1 }); // 查询自定义角色
db.runCommand({ rolesInfo: 1, showBuiltinRoles: true }); // 包含内置角色
db.runCommand({ rolesInfo: "<rolename>" }); // 指定角色
db.runCommand({ rolesInfo: { role: "<rolename>", db: "<database>" } }; // 查询其他数据库中指定角色
```

## 创建用户

```shell
db.createUser({user:"myroot",pwd:"123456",roles:["root"]}); // 创建
db.system.users.find();

db.changeUserPassword("myroot", "123456"); // 修改密码

db.auth("myroot","123456"); // 认证
```
