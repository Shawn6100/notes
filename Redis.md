# Redis常用五大数据类型

## Redis键（key）

|     命令      |                           作用                            |
| :-----------: | :-------------------------------------------------------: |
| set key value |                    向Redis中添加键值对                    |
|    keys *     |                     查看当前库所有key                     |
|  exists key   |                    判断某个key是否存在                    |
|   type key    |                       查看key的类型                       |
|    del key    |                     删除指定的key数据                     |
|  unlink key   |                 删除指定的key（异步删除）                 |
| expire key 60 |                  设置key的过期时间（秒）                  |
|    ttl key    | 查看key还有多久过期<br />（-1表示永不过期，-2表示已过期） |



## 基本命令

|   命令   |              作用              |
| :------: | :----------------------------: |
| select 1 | 切换数据库（数据库下标从0-15） |
|  dbsize  |    查看当前数据库key的数量     |
| flushdb  |           清空当前库           |
| flushall |           清空全部库           |



## Redis字符串（String）

> String 是 Redis 最基本的类型。
>
> 它是二进制安全的，即意味着它可以包含任何数据，如图片或序列化的对象。
>
> 一个Redis 字符串 value 的空间最大是512M

