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
