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




