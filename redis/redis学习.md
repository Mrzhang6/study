# redis学习

## Nosql概述

## redis入门

## 性能测试

## 基础知识

## 五大数据类型



### redis-key

```shell
127.0.0.1:6379> keys * #查看所有的key
1) "name"
2) "sex"
127.0.0.1:6379> get sex
"\xe7\x94\xb7"
127.0.0.1:6379> EXISTS name #判断当前key是否存在
(integer) 1
127.0.0.1:6379> move sex 1 #移除当前key到别的库
(integer) 1
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> select 1 #切换数据库
OK
127.0.0.1:6379[1]> keys *
1) "sex"
127.0.0.1:6379[1]> select 0
OK
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> expire name 10 #设置key的过期时间
(integer) 1
127.0.0.1:6379> ttl name #查看当前key的剩余时间
(integer) 6
127.0.0.1:6379> ttl name
(integer) -2

```

查看不同key的类型

```shell
127.0.0.1:6379> type name 
string
```

### String(字符串)

 ```shell
127.0.0.1:6379> append name hello #往某个key追加字符串，如果key不存在，相当于set
(integer) 7
127.0.0.1:6379> get name
"zdhello"
127.0.0.1:6379> strlen name #获取key长度
(integer) 7


127.0.0.1:6379> set views 0
OK
127.0.0.1:6379> get views
"0"
127.0.0.1:6379> incr views #自增1
(integer) 1
127.0.0.1:6379> incr views
(integer) 2
127.0.0.1:6379> get views
"2"
127.0.0.1:6379> decr views #自减1
(integer) 1
127.0.0.1:6379> incrby views 10 #设置步长
(integer) 11
127.0.0.1:6379> decrby views 5
(integer) 6
 ```

#### 获取字符串范围（getrange)



```shell
127.0.0.1:6379> flushall #清空数据库
OK
127.0.0.1:6379> set key1 hello,zd
OK
127.0.0.1:6379> get key1
"hello,zd"
127.0.0.1:6379> getrange key1 0 3 #截取字符串，闭区间
"hell"
127.0.0.1:6379> getrange key1 0 -1 #获取全部字符串
"hello,zd"
127.0.0.1:6379> 

```



#### 替换字符串(setrange)

```shell
127.0.0.1:6379> set key2 abcdefgh
OK
127.0.0.1:6379> get key2
"abcdefgh"
127.0.0.1:6379> setrange key2 1 000 #替换指定位置开始的字符串
(integer) 8
127.0.0.1:6379> get key2
"a000efgh"

```



#### set时设置过期时间（setex)

```shell
127.0.0.1:6379> setex key3 20 30 #设置时加上过期时间，20s过期
OK
127.0.0.1:6379> ttl key3 #查看key的剩余时间
(integer) 15
127.0.0.1:6379> ttl key3
(integer) 12
127.0.0.1:6379> ttl key3
(integer) 10

```



#### 不存在在set（setnx)

```shell
127.0.0.1:6379> setnx mykey redis #不存在创建mykey
(integer) 1
127.0.0.1:6379> keys *
1) "key2"
2) "key1"
3) "mykey"
127.0.0.1:6379> setnx mykey zds #之前存在mykey，所以返回0
(integer) 0

```

#### 批量设置和读取（mset和mget）

```shell
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"

```

#### 对象

```shell
127.0.0.1:6379> set user:1 {name:zd,age:23,sex:man} #设置一个user:1对象，值为json字符存储这个对象
OK
127.0.0.1:6379> keys *
1) "user:1"
2) "k3"
3) "k1"
4) "k2"
127.0.0.1:6379> get user:1
"{name:zd,age:23,sex:man}"

#巧妙设计 user:{id}:{filed}
127.0.0.1:6379> mset user:2:name zd user:2:age 23
OK
127.0.0.1:6379> keys *
1) "user:2:name"
2) "user:2:age"
3) "k1"
4) "k3"
5) "k2"
6) "user:1"
127.0.0.1:6379> mget user:2:name user:2:age
1) "zd"
2) "23"
127.0.0.1:6379> 

```

#### 先get后set（getset）

```shell
127.0.0.1:6379> getset name zd #如果不存在，则返回nil
(nil)
127.0.0.1:6379> get name
"zd"
127.0.0.1:6379> getset name zs #如果存在则返回原来的值，并设置新的值
"zd"
127.0.0.1:6379> get name
"zs"
127.0.0.1:6379> 

```

>String类似的使用场景:value除了是字符串,还可以是数字
>
>- 计数器
>- 统计数量
>- 粉丝数
>- 对象缓存存储

### List

> 在redis中可以吧list，改为栈，队列等其他数据结构

> 所有的list命令都是l开头的

#### lpush

![image-20201209192656020](D:\学习\redis\imges\image-20201209192656020.png)

#### rpush

![image-20201209192843400](C:\Users\11264\AppData\Roaming\Typora\typora-user-images\image-20201209192843400.png)

```bash
127.0.0.1:6379> lpush list one two #将一个值或多个值放在列表的头部
(integer) 2
127.0.0.1:6379> keys *
1) "list"
127.0.0.1:6379> get list
(error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> lget list
(error) ERR unknown command `lget`, with args beginning with: `list`, 
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> lange list 0 1
(error) ERR unknown command `lange`, with args beginning with: `list`, `0`, `1`, 
127.0.0.1:6379> lange list 0 0
(error) ERR unknown command `lange`, with args beginning with: `list`, `0`, `0`, 
127.0.0.1:6379> lpush list there
(integer) 3

127.0.0.1:6379> lrange list 0 1
1) "there"
2) "two"
127.0.0.1:6379> lrange list 1 1
1) "two"
127.0.0.1:6379> lrange list 0 0
1) "there"
127.0.0.1:6379> 

127.0.0.1:6379> rpush list right #将元素放在列表的尾部
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "there"
2) "two"
3) "one"
4) "right"
127.0.0.1:6379> 
```

#### lrange

获取list中的值

#### lpop（左移除）rpop（右移除）

```bash
127.0.0.1:6379> clear
127.0.0.1:6379> lpush list one two there
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "there"
2) "two"
3) "one"
127.0.0.1:6379> lpop list
"there"
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> rpop list
"one"
```

#### Lindex

```shell
127.0.0.1:6379> lpush list 1 2 3 4
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "4"
2) "3"
3) "2"
4) "1"
127.0.0.1:6379> lindex list 1 #通过下标获取list中某一个值
"3"
127.0.0.1:6379> lindex list 3
"1" 
```

#### llen（list长度）

```bash
127.0.0.1:6379> lpush list 1 2 3 4
(integer) 4
127.0.0.1:6379> llen list
(integer) 4

```

#### lrem(移除指定的值)

```shell
127.0.0.1:6379> lrem list 1 1 #移除list集合中指定个数的value
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "4"
2) "3"
3) "2"
4) "1"
127.0.0.1:6379> lpush list 3
(integer) 5
127.0.0.1:6379> lrange list 0 -1
1) "3"
2) "4"
3) "3"
4) "2"
5) "1"
127.0.0.1:6379> lrem list 2 3
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "4"
2) "2"
3) "1"

```

#### ltrim（截取）

```shell
127.0.0.1:6379> lpush mylist hello1
(integer) 1
127.0.0.1:6379> lpush mylist hello2
(integer) 2
127.0.0.1:6379> lpush mylist hello3
(integer) 3
127.0.0.1:6379> lrange list 0 -1
(empty array)
127.0.0.1:6379> lrange mylist 0 -1
1) "hello3"
2) "hello2"
3) "hello1"
127.0.0.1:6379> ltrim mylist 1 2
OK
127.0.0.1:6379> lrange mylist 0 -1
1) "hello2"
2) "hello1"
127.0.0.1:6379> lpush mylist 1 2 3
(integer) 5
127.0.0.1:6379> lrange mylist 0 -1
1) "3"
2) "2"
3) "1"
4) "hello2"
5) "hello1"
127.0.0.1:6379> ltrim mylist 1 2 #通过下标截取指定长度，list已经改变，只剩截取的元素
OK
127.0.0.1:6379> lrange mylist 0 -1
1) "2"
2) "1"

```

#### rpoplpush

移除列表得到最后一个元素并移入新的列表中

```shell
127.0.0.1:6379> lrange mylist 0 -1
1) "2"
2) "1"
127.0.0.1:6379> rpoplpush mylist myotherlist
"1"
127.0.0.1:6379> lrange mylist 0 -1
1) "2"
127.0.0.1:6379> lrange myotherlist 0 -1
1) "1"

```

#### lset

将列表中指定下标的值替换为另一个值 

```shell
127.0.0.1:6379> exists list
(integer) 0
127.0.0.1:6379> exists mylist
(integer) 1
127.0.0.1:6379> lrange mylist 0 -1
1) "2"
127.0.0.1:6379> lset list 0 1
(error) ERR no such key
127.0.0.1:6379> lset mylist 0 1
OK
127.0.0.1:6379> lrange   mylist 0 -1
1) "1"

```

#### linsert

将某个具体的value插入列表中的前面或者后面

```shell
127.0.0.1:6379> lpush list one two four
(integer) 3
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "two"
3) "one"
127.0.0.1:6379> linsert list two there
(error) ERR wrong number of arguments for 'linsert' command
127.0.0.1:6379> linsert list before two there
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "there"
3) "two"
4) "one"
127.0.0.1:6379> linsert list after one zero
(integer) 5
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "there"
3) "two"
4) "one"
5) "zero"

```

### Set

> set中的值是不能重复的

#### sadd

#### smembers

#### sismember

```shell
127.0.0.1:6379> sadd myset hello #set集合中添加元素
(integer) 1
127.0.0.1:6379> sadd myset zd
(integer) 1
127.0.0.1:6379> sadd myset love
(integer) 1
127.0.0.1:6379> smembers myset #查看set中的所有值
1) "zd"
2) "love"
3) "hello"
127.0.0.1:6379> sismember myset hello #判断set中是否有某个值
(integer) 1
127.0.0.1:6379> sismember myset word
(integer) 0
```

#### scard（set中元素的个数）

```shell
127.0.0.1:6379> scard myset
(integer) 3
```

#### srem(移除指定元素)

```shell
127.0.0.1:6379> srem myset hello
(integer) 1
127.0.0.1:6379> scard myset
(integer) 2
127.0.0.1:6379> smembers myset
1) "zd"
2) "love"
```

#### spop(随机移除元素)

```shell
127.0.0.1:6379> spop myset 
"zd"
127.0.0.1:6379> spop myset 
"love"
127.0.0.1:6379> 
```

#### smov(将一个集合元素移动到另一个)

```shell
127.0.0.1:6379> smove mtset1 myset2 2
(integer) 1
127.0.0.1:6379> smembers myset2
1) "2"
2) "4"
3) "5"
4) "6"
```

> 微博，B站等共同关注！（并集）
>
> - 并集
> - 交集
> - 差集

#### sdiff（差集）

#### sinter（交集）

#### sunion(并集)

```shell
127.0.0.1:6379> smembers myset2
1) "2"
2) "4"
3) "5"
4) "6"
127.0.0.1:6379> sdiff mtset1 myset2
1) "1"
2) "3"
127.0.0.1:6379> sinter mtset1 myset2  
1) "2"
127.0.0.1:6379> sunion mtset1 myset2
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
```

### Hash

> Map集合，key-map，命令以h开头

#### hset

#### hget

#### hmset（设置多个字段）

#### hmget（获取多个字段）

#### hgetall(获取所有字段)

```shell
127.0.0.1:6379> hset myhash name zd sex man age 20
(integer) 3
127.0.0.1:6379> keys *
1) "myhash"
127.0.0.1:6379> hget myhash name
"zd"
127.0.0.1:6379> hmget myhash name age
1) "zd"
2) "20"
127.0.0.1:6379> hgetall myhash
1) "name"
2) "zd"
3) "sex"
4) "man"
5) "age"
6) "20"
127.0.0.1:
```

#### hdel（删除某个字段）

```shell
127.0.0.1:6379> hdel myhash name age
(integer) 2
127.0.0.1:6379> hgetall myhash
1) "sex"
2) "man"
```

#### hlen (获取hash字段数)

```shell
127.0.0.1:6379> hlen myhash
(integer) 1
```

#### hexists(判断hash中指定key是否存在)

```shell
127.0.0.1:6379> hexists myhash sex
(integer) 1
```

#### hkeys(只获取所有字段)

#### hvals(只获取所有值)

### Zset（有序集合）

#### zdd

#### zrange

```shell
127.0.0.1:6379> zadd myset 1 0ne 2 two #添加一个多个值
(integer) 2
127.0.0.1:6379> zrange myset 0 -1
1) "0ne"
2) "two"
```

```shell
127.0.0.1:6379> zadd salary 2500 xiaohong 2000 zd 3000 zs
(integer) 3
127.0.0.1:6379> zrangebyscore salary -inf +inf #从小到大排序
1) "zd"
2) "xiaohong"
3) "zs"
127.0.0.1:6379> zrangebyscore salary +inf -inf
(empty array)
127.0.0.1:6379> zrangebyscore salary +inf -inf withscores
(empty array)
127.0.0.1:6379> zrangebyscore salary -inf +inf withscores
1) "zd"
2) "2000"
3) "xiaohong"
4) "2500"
5) "zs"
6) "3000"

```

#### zcard（获取有序集合中个数）

> 案例：
>
> - 班级成绩表，工资表排序
> - 带权重判断 

## 三种特殊数据类型



### geosptial（地理位置）

>朋友定位，附近的人，打车距离计算？
>
>可以推算地理位置信息

#### getadd（添加地理位置）

```shell
#两极无法添加，一般会下载城市数据通过java程序一次性导入
#key 维度 经度 名称
127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 106.50 29.53 chongqing 114.05 22.52 shenzhen
(integer) 2
127.0.0.1:6379> geoadd china:city 120.16 30.24 hangzhou 108.96 34.26 xian
(integer) 2
```

#### geopos(获取指定城市经纬度)

```shell
127.0.0.1:6379> geopos china:city beijing
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
```

#### geodist（或得给定位置之间距离）

单位：m,km,ml英里，ft英尺

```shell
127.0.0.1:6379> geodist china:city beijing shanghai km
"1067.3788"
127.0.0.1:6379> geodist china:city beijing chongqing
"1464070.8051"
127.0.0.1:6379> geodist china:city beijing chongqing km
"1464.0708"
```

>我附近的人？
>
>获取定位，然后通过半径查询

#### georadius（以给定经纬度为中心，找出某一半径内元素）

```shell
127.0.0.1:6379> georadius china:city 110 30 500 km
1) "chongqing"
2) "xian"
127.0.0.1:6379> georadius china:city 110 30 500 km withcoord withdist count 2
1) 1) "chongqing"
   2) "341.9374"
   3) 1) "106.49999767541885376"
      2) "29.52999957900659211"
2) 1) "xian"
   2) "483.8340"
   3) 1) "108.96000176668167114"
      2) "34.25
```

####  georadiusbymember以元素为中心，找出某一半径内元素

```shell
127.0.0.1:6379> georadiusbymember china:city hangzhou 1000 km
1) "hangzhou"
2) "shanghai"
127.0.0.1:6379> 
```

>zeo 的实现原理为zset,因此可以用zset命令操作。
>
>```shell
>127.0.0.1:6379> zrange china:city 0 -1
>1) "chongqing"
>2) "xian"
>3) "shenzhen"
>4) "hangzhou"
>5) "shanghai"
>6) "beijing"
>127.0.0.1:6379> zrem china:city xian
>(integer) 1
>127.0.0.1:6379> zrange china:city 0 -1
>1) "chongqing"
>2) "shenzhen"
>3) "hangzhou"
>4) "shanghai"
>5) "beijing"
>```
>
>



### Hyperloglog

>什么是基数？不重复的元素

基数统计的算法

优点：占用内存固定，0.81%错误率

```shell
127.0.0.1:6379> pfadd mykey a b c d e f g h i #创建第一组元素
(integer) 1
127.0.0.1:6379> pfcount mykey #统计mykey基数数量
(integer) 9
127.0.0.1:6379> pfadd mykey2 1 2 3 4 5 6 a b c 7 8 9 
(integer) 1
127.0.0.1:6379> pfcount mykey2
(integer) 12
127.0.0.1:6379> pfmerge mykey3 mykey mykey2 #并集 合并两组到mykeys
OK
127.0.0.1:6379> pfcount mykey3
(integer) 18

```

### Bitmaps

> 位存储

统计用户信息,活跃,不活跃!登录,未登录!打卡,两个状态的都可以使用bitmaps

 ```shell
127.0.0.1:6379> setbit sign 0 1 
(integer) 0
127.0.0.1:6379> setbit sign 0 1 
(integer) 1
127.0.0.1:6379> setbit sign 1 1
(integer) 0
127.0.0.1:6379> getbit sign 0
(integer) 1

 ```

统计操作

127.0.0.1:6379> bitcount sign 
(integer) 3



## 事务

> 要么同时成功，要么同时失败，redis单条命令保证原子性，但是事务不保证原子性
>
> Redis事务没有隔离级别概念
>
> 所有的命令在事务中，并没有直接执行！只有发起执行命令才会执行Exec

Redis事务本质：一组命令的集合，一个事务中所有命令会被序列化，在事务执行过程中，会按照顺序执行。

一次性、顺序性、排他性

redis事务：

 - 开启事务（multi）

 - 命令入队（...）

 - 执行事务（exec）

   > 正常执行事务！

```shell
127.0.0.1:6379> multi #开启事务
OK
#命令入队
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec #执行事务
1) OK
2) OK
3) "v2"
4) OK
```

> 放弃事务！

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2 
QUEUED
127.0.0.1:6379> set k4 v4
QUEUED
127.0.0.1:6379> discard #放弃事务
OK
127.0.0.1:6379> get k4
(nil)
```

> 编译型异常（代码有问题，命令有错），事务中所有命令都不会被执行。

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> set k3 v3
OK
127.0.0.1:6379> getset 11 #错误的命令
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> set k5 v5
OK
127.0.0.1:6379> exec #执行事务报错，所有命令都不会执行
(error) ERR EXEC without MULTI
```





> 运行时异常 1/0，执行命令时其他命令可以正常执行。错误命令抛出异常

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incr k1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> set k3 v3
QUEUED
127.0.0.1:6379> exec #虽然第一条命令报错，但其他依旧执行成功
1) (error) ERR value is not an integer or out of range
2) OK
3) OK
127.0.0.1:6379> get k2
"v2"
```



>监控！Watch

悲观锁：

- 很悲观，认为什么时候都会出现问题，无论做什么都会加锁。

乐观锁：

- 认为什么时候都不会出现问题，所以不会上锁，更新数据时判断一下，在此期间是否有人修改过这个数据。version！
- 获取version
- 更新时比较version

> redis监视测试！

正常执行成功！

```bash
127.0.0.1:6379> clear
127.0.0.1:6379> set money 100
OK
127.0.0.1:6379> set out 0
OK
127.0.0.1:6379> watch money #监视money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 20
QUEUED
127.0.0.1:6379> incrby out 20
QUEUED
127.0.0.1:6379> exec #事务正常结束
1) (integer) 80
2) (integer) 20
```

测试多线程修改值，使用watch可以当做乐观锁操作。

```shell
127.0.0.1:6379> get money
"1000"
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> decrby money 20
QUEUED
127.0.0.1:6379> exec #执行之前，另外一个线程修改money值，会导致事务执行失败。
(nil)
```

 

## JRedis

Jredis是官方推荐的java连接开发工具。

1. 导入对应的依赖

```XML
<!--    导入jedis包-->
    <dependencies>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.2.0</version>
        </dependency>
<!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.62</version>
        </dependency>
    </dependencies>
```

2. 编码测试
   - 连接数据库
   - 操作命令
   - 断开连接

```java
public class Test {
    public static void main(String[] args) {
        //1. new Jedis 对象
        Jedis jedis = new Jedis("192.168.149.98", 6379);

        System.out.println(jedis.ping());

    }
```

输出

![image-20201211143520777](D:\学习\redis\imges\image-20201211143520777.png)

### 

## Springboot整合

SpringBoot操作数据：springdata jpa jdbc mongodb redis

SpringData也是和springboot起名的框架

> 整合

在springBoot2.x之后，原来使用的jedis，替换为lettuce

区别：

jedis：采用的是直连，多个线程操作不安全，如果想要避免不安全，使用jedis pool连接池

lettuce：采用netty，实例可以在多个线程中共享，不存在线程不安全的情况。可以减少线程数量，更像Nio模式

源码分析

```java
 @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)//可以自定义来替换默认的
//默认的redisTemplate没有过多的设置，redis对象都是需要序列化的
//两个泛型都是object类型，我们需要强制转换，<String,object>
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(RedisConnectionFactory.class)
//string是redis中最常使用的，所以单独提出一个bean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
```



> 整和测试一下

1. 导入依赖

   ```xml
     <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-data-redis</artifactId>
      </dependency>
   ```

   

2. 配置连接

   ```properties
   #配置redis
   spring.redis.host=192.168.149.98 #主机
   spring.redis.port=6379
   ```

   

3. 测试

   ```java
   package com.zd.redis_pingnoot;
   
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.data.redis.connection.RedisConnection;
   import org.springframework.data.redis.core.RedisTemplate;
   
   @SpringBootTest
   class RedisPingnootApplicationTests {
   
       @Autowired
       private RedisTemplate redisTemplate;
   
       @Test
       void contextLoads() {
           //RedisTemplate 操作不同的数据类型，api和指令一样
           //opsForValue 操作字符串，类似string
           //opsForList 操作list，类似list
           //opsForSet
           //opsForHash
           //opsForGeo
           //opsForZSet......
   
           //除了基本的操作，常用的方法可以直接通过RedisTemplate操作，如事务和基本crud
   
           //获取连接对象，
   //        RedisConnection connection = redisTemplate.getConnectionFactory().getConnection();
   //        connection.flushAll();
   //        connection.flushDb();
   
           redisTemplate.opsForValue().set("mykey","Hi好");
           System.out.println(redisTemplate.opsForValue().get("mykey"));
       }
   
   }
   
   ```

   ![image-20201211165713104](D:\学习\redis\imges\image-20201211165713104.png)



默认序列化是用jdk序列化，我们可能用json序列化

![image-20201211165855088](D:\学习\redis\imges\image-20201211165855088.png)

关于对象的保存，保存之前需要先序列化，否则会报错。

![image-20201211171605664](D:\学习\redis\imges\image-20201211171605664.png)



## redis.conf详解



启动的时候就通过配置文件来启动



> 单位

![image-20201211183920424](D:\学习\redis\imges\image-20201211183920424.png)

1. 配置文件对unit单位大小写不敏感

> 包含

![image-20201211184150887](D:\学习\redis\imges\image-20201211184150887.png)

可以包含多个配置文件

> 网络

```bash
bind 127.0.0.1 #绑定ip
protected-mode yes #受保护模式
port 6379 #端口
```



> 通用配置

```bash
daemonize yes #以守护进程的方式进行，默认是no
pidfile /var/run/redis_6379.pid #如果以后台方式文件，需要制定pid进程文件

# 日志
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)生产环境使用
# warning (only very important / critical messages are logged)
loglevel notice
logfile " #日志的文件位置明

databases 16 #数据库数量
always-show-logo yes #是否总是显示logo
```

> 快照

持久化，在规定时间内，执行了多少次操作，则会持久化到文件 .rdb .aof

redis是内存数据库，没有持久化会断电即失。

```shell
#如果900s内，至少有一个key进行修改，就进行持久化操作
save 900 1
#如果300s内，至少有10key进行修改，就进行持久化操作
save 300 10
#如果60s内，至少有一个10000进行修改，就进行持久化操作
save 60 10000
#之后可以自己定义

stop-writes-on-bgsave-error yes #持久化出错，是否会继续工作
rdbcompression yes #是否压缩rdb文件，需要消耗一些cpu资源

rdbchecksum yes #保存rdb文件时，进行错误的检查校验

dir ./ #rdb文件保存的目录

```

> REPLICATION 复制,在后面主从复制



> SECURITY 安全

```shell
#可以设置redis密码，默认是没有密码的
127.0.0.1:6379> config get requirepass #获取redis密码
1) "requirepass"
2) ""
127.0.0.1:6379> config set requirepass 123456 #设置redis密码
OK
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "123456"
127.0.0.1:6379> exit
[zd@localhost ~]$ redis-cli -p 6379
127.0.0.1:6379> ping
(error) NOAUTH Authentication required.  
127.0.0.1:6379> auth 123456 #auth登录
OK
127.0.0.1:6379> ping
PONG
```



> 限制clients

```shell
maxclients 10000 #设置redis最大的客户端数

maxmemory <bytes> #redis 配置的最大内存容量

maxmemory-policy noeviction #内存达到上限的处理策略
#移除一些过期的key等等
- noeviction：当内存使用达到阈值的时候，所有引起申请内存的命令会报错。
- allkeys-lru：在所有键中采用lru算法删除键，直到腾出足够内存为止。
- volatile-lru：在设置了过期时间的键中采用lru算法删除键，直到腾出足够内存为止。
- allkeys-random：在所有键中采用随机删除键，直到腾出足够内存为止。
- volatile-random：在设置了过期时间的键中随机删除键，直到腾出足够内存为止。
- volatile-ttl：在设置了过期时间的键空间中，具有更早过期时间的key优先移除。
```



> APPEND ONLY MODE aof配置

```shell
appendonly no #默认不开启，默认是使用rdb方式持久化的，在大部分情况下rdb完全够用了
appendfilename "appendonly.aof" #持久化文件名字

# appendfsync always #每次修改都会sync，性能消耗
appendfsync everysec #每秒执行一次sync，可能会丢失这1s数据
# appendfsync no     #不执行sync，这个时候操作系统自己操作数据，速度最快

```

## **redis持久化**

> redis是内存数据库，如果不将内存中的数据库保存到磁盘，一旦服务器进程退出，服务器中的数据库状态也会消失。所以redis提供了持久化功能。