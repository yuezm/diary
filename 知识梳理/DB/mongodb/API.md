# 操作

## 基本操作

```mongo
show dbs # 查看全部数据库
use $DB_Name # 使用某个数据库，如果不存在则创建
db.dropDatabase() # 删除数据库

show Collections # 查询所有表

db.createCollection(collection)
db.collection.drop() # 删除表
```

## 增

```mongo
db.collection.insert($DATA, options) # 新增一条记录
db.collection.insertOne($DATA, options) # 新增一条记录
db.collection.insertMany([$DATA], options) # 多条记录

options:
  writeConcern: document
  ordered: boolean
```

## 删

```mongo
db.collection.remove($Filter) # 切记骚操作，给删完了
```

## 改

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

## 查

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

### query

```mongo
$Query

1\. 对象 { filed: xx }
2\. 正则 { filed: /xx/ }
3\. 各种的操作符，列举几个常用的
  $gt $gte $lt $lte $eq $ne
  $in $nin
  $and $or $nor $not
```
