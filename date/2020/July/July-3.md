# July-3

## Mongodb

### 特点

1. 灵活的文档模型
2. 高可用（副本集，切片）
3. 高性能
4. 高扩展
5. BSON

### 数据对象

1. DB：数据库
2. Collection：数据库中的表
3. Document：表的一条数据
4. Filed：一条数据中的一列

### 操作

#### 基本操作

```mongo
show dbs # 查看全部数据库
use $DB_Name # 使用某个数据库，如果不存在则创建
db.dropDatabase() # 删除数据库

show Collections # 查询所有表

db.createCollection(collection)
db.collection.drop() # 删除表
```

#### 增

```mongo
db.collection.insert($DATA, options) # 新增一条记录
db.collection.insertOne($DATA, options) # 新增一条记录
db.collection.insertMany([$DATA], options) # 多条记录

options:
  writeConcern: document
  ordered: boolean
```

#### 删

```mongo
db.collection.remove($Filter) # 切记骚操作，给删完了
```

#### 改

```mongo
db.collection.update($Filter, $Update, options)
db.collection.updateOne($Filter, $Update, options)
db.collection.updateMany($Filter, $Update, options)
db.collection.replaceOne($Filter, $Update, options)

# 这是 update 的 options，包含属性最多，以他为列举
options
  upsert: boolean, // 如果为true，未找到则插入；false 未找到则不插入，默认 false
  multi: boolean, // true 则更新多个文档，默认 false
  writeConcern: document, // 写存在问题的话，抛出问题级别
  collation: document, //
  arrayFilters: [ filterdocument1, ... ],
  hint: document|string // Available starting in MongoDB 4.2


db.collection.save($DATA, options) # 如果存在则更新，否则为插入
```

#### 查

```mongo
db.collection.find($Query, $projection)
db.collection.findAndModify({
  query: <document>,
  sort: <document>,
  remove: <boolean>,
  update: <document or aggregation pipeline>, // Changed in MongoDB 4.2
  new: <boolean>,
  fields: <document>,
  upsert: <boolean>,
  bypassDocumentValidation: <boolean>,
  writeConcern: <document>,
  collation: <document>,
  arrayFilters: [ <filterdocument1>, ... ]
})
db.collection.findOne($Query, $projection)
db.collection.findOneAndDelete($Query, $projection)
db.collection.findOneAndUpdate($Query, $projection)
db.collection.findOneAndReplace($Query, $projection)

projection
  - filed: 1、true 返回，0、false 不返回
  -

db.collection.find()
  .limit() # 显示返回条数
  .skip() # 跳过多少条
  .sort({ filed: 1, filed: -1 }) # 排序
  .count() 统计

db.collection.count($Query) # 统计

```

##### query

```mongo
$Query

1. 对象 { filed: xx }
2. 正则 { filed: /xx/ }
3. 各种的操作符，列举几个常用的
  $gt $gte $lt $lte $eq $ne
  $in $nin
  $and $or $nor $not

```

[Query Selectors](https://docs.mongodb.com/manual/reference/operator/query/)

[projection](https://docs.mongodb.com/manual/reference/method/db.collection.findOne/#findone-projection)

### 索引

索引是特殊的数据结构，它以易于遍历的形式存储集合数据集的一小部分。索引存储特定字段或一组字段的值，按字段值排序。索引项的排序支持有效的相等匹配和基于范围的查询操作。MongoDB 索引使用 B 树数据结构（确切的说是 B-Tree，MySQL 是 B+Tree）

#### 地理空间索引

Geospatial Index：为了支持对地理空间坐标数据的有效查询，MongoDB 提供了两种特殊的索引：返回结果时使用**平面几何的二维索引**和返回结果时使用**球面几何的二维球面索引**。

#### 文本索引

Text Indexes：MongoDB 提供了一种文本索引类型，支持在集合中搜索字符串内容。这些文本索引不存储特定于语言的停止词（例如“the”、“a”、“or”），而将集合中的词作为词干，只存储根词。

#### 哈希索引

Hashed Indexes：为了支持基于散列的分片，MongoDB 提供了散列索引类型，它对字段值的散列进行索引。这些索引在其范围内的值分布更加随机，但只支
持相等匹配，不支持基于范围的查询

#### 索引操作

```mongo
db.collection.getIndexes() # 查询索引
db.collection.createIndex(keys, options) # 创建索引
db.collection.dropIndex(index)
db.collection.dropIndexes()

key：包含字段和值对的文档，其中字段是索引键，值描述该字段的索引类型。对于字段上的升序索引，请
指定值 1；对于降序索引，请指定值-1。比如： {字段:1 或-1} ，其中 1 为指定按升序创建索引，如果你
想按降序来创建索引指定为 -1 即可。另外，MongoDB 支持几种不同的索引类型，包括文本、地理空
间和哈希索引

options：
  background Boolean 建索引过程会阻塞其它数据库操作，background 可指定以后台方式创建索引，即增加"background" 可选参数。 默认值为 false。

  unique Boolean 建立的索引是否唯一。指定为 true 创建唯一索引。默认值为 false.

  name string 索引的名称。如果未指定，MongoDB 的通过连接索引的字段名和排序顺序生成一个索引名
称。

  partialFilterExpression document 3.0

  sparse Boolean 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为 true 的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 false.

  expireAfterSeconds integer 指定一个以秒为单位的数值，完成 TTL 设定，设定集合的生存时间

  hidden Boolean

  storageEngine document
```

### 副本集

副本集：维护相同 mongod 数据集的服务，副本集提供冗余和高可用

1. 在不同数据库服务器上提供多个数据副本，提供一定的容错能力，防止单个服务器数据丢失
2. 副本集包含**数据承载节点**和**仲裁节点**。
   - 承载数据节点：在承载数据的节点中，存在一个主节点和多个副本节点，主节点接收所有操作，副本节点不能进行操作（但可以配置为只读）。
   - 仲裁节点：选举节点，当主节点挂了，仲裁节点从副本集内部选举一个新的主节点

**副本集和主从集群区别**

1. 副本集没有固定的主节点，主节点是由副本节点选举出来的
2. 整个副本集只存在一个主节点和多个副本节点，当前主节点挂了，从剩余的副本节点继续选举

#### 副本集角色

1. Primary: 主节点，接收所有读写操作
2. Secondary: 副本节点，通过复制主节点的操作以维护相同的数据，不可操作，但可以配置为只读
3. Arbiter: 仲裁者，不保留任何数据，只具有投票选举。**也可以把仲裁者维护成副本节点**，这样一个节点就同时存在两个角色，可以理解为一个副本节点，同时具有仲裁功能

##### 选举

**选举条件**：

1. 主节点故障
2. 主节点网络不可达，默认心跳为 10s
3. 人工干预 rs.stepDown(600);

**选举原则**：

1. 票数最高，并且必须要获得大部分成员的投票。**什么叫大部分成员？**至少大于一半。例如三个投票成员，则大部分至少是 2。当复制集内存活的成员不足大多数时，则整个副本集无法选举出主节点，则服务降级，只提供读取服务
2. 如果票数相同，则数据新的节点获胜，数据的新旧是通过操作日志 oplog 来对比的。

数据比较时，节点优先级非常重要，优先级可以理解为票数，(0-1000)，如果选举节点优先级为 1000，则它一次投票就为 1000。主节点和其他副节点优先级默认都是 1，选举节点必须是 0，（即不具备选举权，但可以投票）

**选举时，请千万注意大多数，如果只存在一个节点的情况下，尽管他获取的票数肯定是最高，但他不可能是大多数，所以会服务降级**！！！

#### 副本集操作

##### 初始化

当连接上数据库时，需要命令是无法使用的，例如 show dbs，需要初始化副本集才可以进行操作

```shell script
rs.initiate(configuration); # configuration 不传则为默认配置
```

[replica-set-configuration-document](https://docs.mongodb.com/manual/reference/replica-configuration/#replica-set-configuration-document)

##### 查看

```shell script
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

##### 配置

```shell
rs.add(host, arbiterOnly); # 添加其他副本节点，arbiterOnly是否为仲裁节点
rs.addArb(host); # 添加仲裁节点
rs.remove(); # 删除

rs.slaveOk(); # 配置副本节点只读为，db.getMongo().setSlaveOk() 简化命令
rs.slaveOk(true); # 配置副本节点只读
rs.slaveOk(false); # 取消副节点的读
```

#### 副本集示例

3 个节点（一主一副一仲裁）

##### 副本成员（PRIMARY）

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

##### 副本成员（SECONDARY）

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

##### 仲裁者

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

## LeetCode

### 三角形最小路径集合

思路：

1. 动态规划 dp[i][j] = min(dp[i-1][j-1], dp[i-1][j]);
2. 完成计算后，取最后一排最小值

```typescript
function minimumTotal(triangle: number[][]): number {
  const row = triangle.length;

  for (let i = 1; i < row; i++) {
    for (let j = 0; j < triangle[i].length; j++) {
      let m1 = Number.MAX_SAFE_INTEGER,
        m2 = Number.MAX_SAFE_INTEGER;

      if (j > 0) {
        m1 = triangle[i - 1][j - 1];
      }

      if (j < i) {
        m2 = triangle[i - 1][j];
      }

      triangle[i][j] += Math.min(m1, m2);
    }
  }
  return Math.min(...triangle[row - 1]);
}
```

[三角形最小路径集合](https://leetcode-cn.com/problems/triangle/)

### 不同的二叉搜索树

思路：动态规划，依次选取每一个数为节点，拿 1、2、3 举例

首先选取 1 为根节点 => `f(0) * f(2)`
首先选取 2 为根节点 => `f(1) * f(1)`
首先选取 3 为根节点 => `f(2) * f(1)`

递推公式：dp(i) = `dp(0) * dp(i-1) + dp(1) * dp(i-2)...dp(n-1) * dp(1)`

```typescript
function numTrees(n: number): number {
  if (n === 0) return 0;
  const store: number[] = [1, 1];
  for (let i = 2; i <= n; i++) {
    let sum = 0;
    for (let j = 1; j <= i; j++) {
      sum += store[j - 1] * store[i - j];
    }
    store[i] = sum;
  }
  return store[n];
}
```

[不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees/)

### 外观数列

思路 1：队列存储，由于这里存在描述数字和真实数字，例如 111 ，描述数字为 3，真实数字为 1，组合为 31，所以需要两个值
思路 2：递归 fn(n) = handler(fn(n-1));

```typescript
// 迭代存储
class Queue {
  store: number[] = [];

  push(v: number): void {
    this.store.unshift(v);
  }

  clear(): void {
    this.store = [];
  }

  isEmpty(): boolean {
    return this.store.length < 1;
  }

  // 由于空repeatPop无意义，所以不做判断
  repeatPop(): { value: number; count: number } {
    let lastIndex = this.store.length - 1;
    const lastNumberValue = this.store[lastIndex];
    let count = 0;

    while (lastIndex >= 0 && lastNumberValue === this.store[lastIndex]) {
      this.store.pop();
      lastIndex--;
      count++;
    }

    return {
      value: lastNumberValue,
      count,
    };
  }
}

function countAndSay(n: number): string {
  let queue = new Queue();
  const tempQueue = new Queue();

  queue.push(1);

  for (let i = 2; i <= n; i++) {
    while (!queue.isEmpty()) {
      const { value, count } = queue.repeatPop();

      tempQueue.push(count);
      tempQueue.push(value);
    }
    queue.store = tempQueue.store;
    tempQueue.clear();
  }

  return queue.store.reverse().join('');
}
```

```go
// 递归
func countAndSay(n int) string {
	if n == 1 {
		return "1"
	}

	prevRes := countAndSay(n - 1)
	res := ""

	j := 1

	prevValue := prevRes[0]
	count := 1

	for {
		if j >= len(prevRes) {
			break
		}

		if prevValue == prevRes[j] {
			count++
			j++
		} else {
			res += strconv.Itoa(count) + string(prevValue)

			prevValue = prevRes[j]
			count = 1
			j++
		}
	}
  
  res += strconv.Itoa(count) + string(prevValue)
	return res
}
```
