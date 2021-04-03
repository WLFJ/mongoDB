# mongoDB

## 导入导出数据

注意对于BSON和其他数据，命令是不同的。

同时导入的时候可以选择删除原有collection之后导入，这样可以避免很多问题。

```shell
mongodump --uri "mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies"

mongoexport --uri="mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" --collection=sales --out=sales.json

mongorestore --uri "mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies"  --drop dump

mongoimport --uri="mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" --drop sales.json
```

## 简单查询（Atlas UI）

命名空间：`数据库名称`.`collection`

简单查询：
```json
{"state": "NY"}
{"state": "NY", "city": "ALBANY"}
```

** 注意查询的时候字段要按照顺序**

## shell中的查询

1. 连接数据库

```shell
mongo "mongodb+srv://<username>:<password>@<cluster>.mongodb.net/admin"
```

2. 连贯操作

```js
show dbs

use sample_training

show collections

db.zips.find({"state": "NY"})
```

在执行`find`之后，只会显示出结果集中的20条数据，使用`it`命令进行遍历。
因为`find`返回一个迭代器。
同时注意其返回数据并不具有顺序性。

3. find其他功能
```js
db.zips.find({"state": "NY"}).count()

db.zips.find({"state": "NY", "city": "ALBANY"})

db.zips.find({"state": "NY", "city": "ALBANY"}).pretty()
```

## 插入数据

```js
db.<collection_name>.insert({
    ...
})
```

插入多条文档

```js
db.<collection_name>.insert([
    {
        ...
    },
    {
        ...
    },
    ...
])
```

如果插入中间出现错误，则错误文档之后的内容不会插入。如果想继续插入，则要指定参数。
```js
db.<collection_name>.insert([
    {
        ...
    },
    {
        ...
    },
    ...
],{ "ordered": false })
```

如果`<collection_name>`所指向的collection不存在，则会同时创建。
同样，如果`use <db_name>`所指向db不存在，在插入时会同时创建。

## 更新文档

可以使用`updateOne`或者`updateMany`.

* $inc - increment
* $set
* $unset
* $push

### Upsert: true

如果查询不到文档，则创建新文档。
其结构会与查询语句的结构相同，可以省去很多查询、创建语句！

## 删除文档、集合

删除文档使用`deleteOne` or `deleteMany`
集合则为`db.<collection_name>.drop()`
数据库不需要删除，当其中没有collection的时候会自动删除

## 高级CRUD

这里需要与`$set`等做区分，其用法大概是这样的：
```json
{ <field_name>: {$<opr>: <args>, ...}, ...}
```
当不添加Query Operators的时候，默认是等于比较。

我们可以理解为给某个field添加“某种”属性！

### Query Operators

* $eq
* $ne
* $lt
* $gt
* $lte
* $gte

### Logical Operators

* $and
* $nor
* $or
```json
{ $<opr>: [{...}, {...}] }
```
* $not
注意其并不能放在前面直接使用，而要在field中！
```json
{ field: { $not: { <operator-expression> } } }
```

需要注意的是查询条件的大括号中要以key开头，而不能直接放条件（这样就不是dict了不是吗！）【做题有感】

### expressive query

有时候我们需要比较文档内的属性，比如说给定一个记录*开始位置*和*结束位置*的文档collection，我们想要统计其中两个位置相同的文档条目，这就需要比较文档内的情况了。

```json
{
    "start pos": "beijing",
    "end pos": "taiyuan"
}
```

对应的查询如下：
```js
db.<collection_name>.find(
    {
        $expr: {
            $eq: ["$start pos", "$end pos"]
        }
    }
)
```

### array query

下面我们研究一下如何查询数组。

```json
{
    {
        friend: [ '张三', '李四', '王五']
    }
}
```

是数组，且非空
```js
{ $match: { writers: { $elemMatch: { $exists: true } } }
```

如果直接查询
```js
{
    friend: '张三'
}
```

将返回所有这个数组中**包含**这个元素的文档

```js
{
    friend: ['张三', '李四']
}
```

将返回所有内容完全一样的文档（内容、顺序都一样）

* $all:[...]
用于*集合*查询

* $size: num
指定数组的大小

* $elemMatch
查找数组中满足条件的元素

例如文档中包含成绩数组，我们想看其中大于90分的子文档（当然返回的结果依旧是完整的文档）

```js
{
    scores: {
        $elemMatch: {
            score: {
                $gt: 90
            }
        }
    }
}
```

### 子文档查询

```json
{
    {
        jobs: [
            {
                company: Intel
            },
            {
                company: AMD
            }
        ]
    }
}
```

如果我们想查看第二个工作在AMD的人，可以这样查询
```js
{
    'jobs.1.campany': 'AMD'
}
```

### projection (投影）

```js
db.<collection_name>.find({MQL}, {projection})
```

### 指定字段是否出现

```json
{
    _id: 0, // 不显示_id
    name: 1 // 显示name
}
```

如果是数组里面感兴趣的子元素呢？(详情见*array query*)
没错！与数组查询一样！
```json
{
    scores:{
        $elemMatch: {
            score: {
                $gt: 90
            }
        }
    }
}
```

## aggregation (聚合）

查询文档已经无法满足我们的需求了。我们希望“创造”一些我们感兴趣的数据；换句话说，我们希望对其进行统计分析。

基础语法：
```js
db.<collection_name>.aggregation([ ...oprs...])
```

## cursor 方法

* `sort({<field_name>: 1 生序 or -1 降序})`
* `limit(限制显示的条数)`
* `pretty()`

## index

```js
db.<collection_name>.createIndex({ ...just..like...sort...})
```

## string operation

* 分割字符串 split:['$field', '<splitter>']
