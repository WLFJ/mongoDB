# mongoDB Aggregation

## $match

* 只能在第一个里面使用`$where`、`$test`
* 第一个使用可以享受index加速

## $project

投影能做什么？
* 显示想要的字段
* 创建新字段
```json
{
    'new_name': '$field.old_name'
}
```
* 进行运算
```json
{
    'calc': {
        $multiply: [ '$field.name', 100],
    }
    'calc1': {
        $devide: [ '$field.name', 100],
    }
}
```

### 在project的collection中reduce

* sum
* min
* max
* avg
* stdDevPop
* stdDevSam

### array 元素转换

```js
db.movies.aggregate([
    {
        $match: {
            writers: {
                $elemMatch: {
                    $exists: true
                },
            }
        },
    },
    {
        $project: {
            _id: 0,
            // 转换['zs (导演)', 'ls (演员）']为['zs', 'ls']
            writers: {
                $map: {
                    // 相当于for循环
                    input: '$writers',
                    as: 'writer',
                    in: {
                        // array[0]
                        $arrayElemAt: [
                            {
                                $split: [ '$$writer', ' (']
                            },
                            0
                        ]
                    }
                }
            },
        }

    }
])
```

## unwind

与`$group`分组不同，unwind恰好是其逆运算

## lookup

左连接。或者亦可以理解为依照一个field在另一个collection中寻找满足条件的文档。

一定要明确“谁找谁”

```js
{
   $lookup:
     {
       from: 要找东西的表（外表）,
       localField: 我自己表上要查找的字段,
       foreignField: 外表上匹配的字段,
       as: 要保存命中文档的filed名字,
     }
}
```

## graphLookUp

```js
{
    // 传入文档
   $graphLookup: {
       // 我们需要指定数据通路
      from: 要查找的文档,
      startWith: 要开始的id,
      connectFromField: <string>, // 这两个
      connectToField: <string>, //   从“箭头”的角度思考
      as: <string>,
      maxDepth: <number>,
      depthField: <string>,// 指定一个field名，用于存储层数
      restrictSearchWithMatch: <document>
   }
}
```

## $sort

## $skip

## $limit

## $count

## $addFields

添加field。但是与`$project`不同。
* 缩短nested field的长度

## group



## $geoNear

地理位置搜索，不感兴趣

## $min

用其他功能实现`$min`
```js
$project: {
    min_value: {
        $reduce: {
            input: '$list', 
            initialValue: -Infinity,
            in: {
                $cond: [
                    { $gt: { '$$this', $$value },
                    '$$value',
                    '$$this'
                ]
            }
        }
    }
}
```

## $sample

返回指定数量随机选择的文档

```js
{ $sample: { size: <positive integer> } }
```
* n <= 5%
* size(collection) < 100
* pipeline 在第一个

如果上面的都不满足，则会使用内存排序（意味着可能达到内存限制）

## facets

这里我们讨论一些将文档分在不同集合的技巧。

### $countBySort

将分量统计并按照个数排序

### $bucket

### $bucket

```js
{
  $bucket: {
      groupBy: 要统计的field,
      boundaries: [ <lowerbound1>, <lowerbound2>, ... ],// 注意前开后闭的区间，注意同种类型，如果数值则可以添加Infinity字段
      default: <literal>,// 没落在区间中的放到默认字段中
      output: {
          // 指定输出的字段
         <output1>: { <$accumulator expression> },
         ...
         <outputN>: { <$accumulator expression> }
         '分类': {
            $addToSet: '$分类field'
         },
      }
   }
}
```

### $bucketAuto

与`$bucket`不同，我们现在只需要指定bucket数量，不用指明区间了。

### $facet

同时生成多个子标签。（可以想象为商城多个分类）

```js
{
  $bucketAuto: {
      groupBy: <expression>,
      buckets: <number>,
      output: {
         <output1>: { <$accumulator expression> },
         ...
      }
      granularity: <string>
  }
}
```

## 注意事项

1. 注意元素的格式是否一致（确保都是数组、内容和我们想得一样）
2. 如果要计算field之间的结果，不能用类似`$divide`这样的运算符，因为其第二个元素是值！要用类似`$avg`
3. 注意其并行计算特性（在同一个project中后面的field不能引用前面的field）
