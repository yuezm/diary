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

### 分片集群

## LeetCode

### 三角形最小路径集合

### 不同的二叉搜索树

### 外观数列
