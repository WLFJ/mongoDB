# mongoDB

## insert和save的区别

两者的操作相同，但是当出现`_id`冲突时:

* `insert`报错
* `save`转为更新

并且两者的结果json也不相同（因为功能不同）

## update

```js
db.<collection_name>.update({匹配条件}, {更新内容}, 
                            没有找到是否插入, 是否更新多条);
```

### 更新内容

如果是已有字段更新，则直接指定内容即可。
```js
{ name: 'zs' }
```

如果要在文档中添加内容
```js
{ 
    $set: {
        sex: 'm',
        age: 18
    }
}
```

### 修改子文档

必须使用`$set`

### 修改数组

必须使用`$set`

```js
[
    {
        arr: [
            {
                name: 'zs',
                age: 18,
            },
            {
                name: 'ls',
                age: 20,
            }
        ]
    }
]
```

```js
{
    $set: {
        // 更新数组中的子文档项目
        "arr.0.name" : 'zs1',
        // 更新数组中整个子文档
        "arr.1" : {
            name: 'ww',
            age: 100
        }
    }
}
```

### 更新运算符

`$set`就是其中之一

#### $inc -> increment

```js
{
    $inc: {
        age: 5 // age += 5
    }
}
```

#### $unset -> 删除字段

```js
{
    $unset: {
        age: 1 // 为1则删除，其他字段不写默认为0
    }
}
```

#### $addToSet -> 在集合中添加元素

```json
{
    books: ['c']
}
```

```js
{
    $addToSet: {
        books: 'java', // 在数组中添加'java', 注意其具有去重性
    }
}
```

#### $push -> 压栈

```json
{
    books: ['c']
}
```

```js
{
    $push: {
        books: 'java', // 在数组后面添加元素
    }
}
```

如果使用不存在的名字，则将自动创建。

#### $pop -> 出栈

```json
{
    books: ['c']
}
```

```js
{
    $pop: {
        books: 1, // 1: 删除最后一个；-1：删除第一个
    }
}
```

#### $pullAll

```json
{
    books: ['c']
}
```

```js
{
    $pullAll: {
        books: ['c', 'java'], // 1: 删除最后一个；-1：删除第一个
    }
}
```

### 批量添加

```json
{
    books: ['c']
}
```

```js
{
    $<update_opr>: {
        books: {
            $each: ["php", "SQL", "javascript"]
        }
    }
}
```

### 数组定位器

```json
{
    books: [
        {
            name: "book1",
            price: 100
        },
        {
            name: "book2",
            price: 200
        },
        {
            name: "book3",
            price: 300
        },
    ]
}
```

```js
// 3种办法,待补充
```

## 删除文档

```js
db.<collection_name>.remove({查找条件}, [{justOne: true}只删除一条])
```

## 分组



## 聚合

## 映射-归并
