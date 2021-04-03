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

#### array 元素转换

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


## 注意事项

1. 注意元素的格式是否一致（确保都是数组、内容和我们想得一样）
