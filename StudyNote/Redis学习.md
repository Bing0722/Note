<div style="text-align: center;"><font size="6">Redis学习</font></div>

## 一、五大数据类型与命令

### 0、常规命令

```r
# 数据库的切换 db是数据库的编号
select [db]

# 清空当前数据库
flushdb

# 清空所有数据库
flushall

# 删除key值
del key

# 查看当前数据库key值的数量
dbsize

# 查看当前数据库有哪些key值
keys *
 
# 把key从当前库移动到目标库 
move key [num]

# 查看key是否存在
EXISTS key

# 设置某个key的过期时间
expire key  seconds   #expire k1 20    #设置k1的存活时间是20s

# 查看key的剩余时间  -1代表永不过期   -2表示已经过期
ttl key

# 查看key的类型
type key
```

### 1、string数据类型

是二进制安全的 可以存放任意类型的数据

```r
# 设置与获取
set k1 value1
get k1

mset k2 value2 k3 value3 k4 value4...
mget k1 k2 k3 ....

# 获取子串
getrange key start end    #如果想找到最后一个可以使用-1
例如：getrange k1 0 4   

# 修改key数据某个位置的数据
setrange key offset value  

# incr/incyby  增加的key的value必须是数值类型
INCR/INCRBY key [num]  对value值加1/加num，value必须是数字
```

### 2、list数据类型
双向链表

```r
# 添加元素有左右之分
lpush/rpush key value1 value2 value3 ....

# 删除头或者尾
lpop/rpop key

# 遍历
lrange key start end

# 支持下标操作的
# 可以使用下标进行获取lindex
lindex  key  [idx]

# 可以使用下标进行设置lset
lset key [idx] newValue
```

### 3、set数据类型
集合 底层使用的是哈希表 也就表明 `set` 中的元素是没有顺序的

```r
# 添加元素的命令
sadd key value1 value2 value3....

# 遍历
smembers key

# 在集合中随机选出num个数
SRANDMEMBER key num 

# 移除并返回集合中的一个/num个随机元素
SPOP key  [num] 
```

### 4、sort set数据类型
可以为每个`key`设置`double`类型的分数进行排序

```r
# 添加元素
zadd key score1 value1  score2 value2 ....

# 遍历元素
zrange key  start end [withscores]
```

### 5、hash数据类型
字典，`Key-value`模式不变 但`value`是一个键值对 等价于`map<key, map<key1, value>>`

```r
# 添加元素
hset key field value

# 获取元素
hget key field 

# 设置与获取多个
hmset/hmget
```

## 二、配置文件

## 三、Redis的持久化

## 四、Redis的事务

## 五、主从复制

## 六、哨兵模式