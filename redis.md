#Redis

## 什么是Redis?
Redis是一个开源的,遵守BSD协议的高性能的key-value数据库。

* 支持数据持久化。
* 提供list,set,zset,hash等数据结构的存储。
* 支持数据的备份(master-slave)模式的数据备份。

## Redis的优势

* 性能极高,读可达到每秒110000次,写可以达到每秒81000次。
* 支持丰富的数据类型。
* Redis的所有操作都是原子的,意思就是要么全部执行成功,要么全部执行失败。单个操作是原子的,多个操作也支持事物。
* 具有丰富的特性-Redis支持publish/subscribe,通知,key过期等特性。

## Redis的安装
先下载tar.gz包,官网为***[redis.io](https://redis.io/)***。

```
$ tar zxvf redis.tar.gz
```
```
$ cd redis-4.0.10/
```
```
$ make
```
启动redis服务。

```
$ redis-server	
```
这种方式启动redis使用的是默认配置,也可以通过参数告诉redis使用指定配置文件启动服务。如:

```
$ redis-server ../redis.conf
```
启动服务之后,可以启动redis-cli去和服务端进行交互了。比如:

```
$ redis-cli
```
进入之后的界面如下：

	127.0.0.1:6379>
测试存入一条数据:

```
127.0.0.1:6379>set name Jerry //返回ok
```
```
127.0.0.1:6379>get name //返回Jerry
```
通过测试看redis是否已经安装成功。

```
127.0.0.1:6379>ping	//返回pong
```
## Redis配置
redis的配置文件位于根目录下,名为redis.conf，可以通过`CONFIG `命令来查看配置。

```
127.0.0.1:6379>CONFIG GET *
```
或获取日志级别：

```
127.0.0.1:6379>CONFIG GET loglevel
-------------
1) "loglevel"
2) "notice"
-------------
```
可以通过`CONFIG SET`或者修改`redis.conf`文件来修改配置。`redis.conf`的配置项如下:

1. `daemonize no`
2. `pidfile /var/run/redis.pid`
3. `port 6379`
4. `bind 127.0.0.1`
5. `timeout 300`,客户端300秒之后关闭连接,如果为0说明关闭此功能
6. `loglevel verbose`,redis有4个日志级别,debug、verbose、notice、warning,默认为verbose
7. `logfile stdout`,日志记录方式,默认为标准输出。如果redis被设置为守护线程,而这里又配为标准输出,则日志将会发送给/dev/null
8. `database 16`,设置数据库的数量,默认为0,可以使用`select <dbid>命令来制定数据库id
9. `save <seconds> <changes>`,指定在多长时间内,有多少次更新操作就将数据同步到数据文件,可以多条件组合。如,redis默认配置文件中提供了3种条件。
	* save 900 1
	* save 300 10
	* save 60 10000  
分别表示900秒内有1次变更,300秒内有10次变更,60秒内有10000次变更。
10. `rdbcompression yes`,制定数据是否进行压缩,可以通过取消来节约cpu时间，但是会导致数据库文件变的巨大。
11.`dbfilename dump.rdb`,指定数据库文件,默认为dump.rdb
12. `dir ./`,指定数据库文件的路径
13. `slaveof <masterip> <masterport>`,当设置本机为slave时,设置masterip和masterport,在Redis启动的时候,会进行同步。
14. `masterauth <master-password>`,当master设有密码的时候,slave配置master的验证密码。
15. `requirepass foobared`,设置redis连接密码,如果设置了,在连接时需要通过`AUTH <password>` 来提供密码。默认关闭。
16.  `maxclients 128`,设置同一时间极大客户连接数,默认无限制。如果超过限制,redis会关闭新的连接并且返回max number of clients reached信息。
17.  `maxmemory <bytes>`,设置最大内存,redis会在启动的时候把数据都加载到内存,清除到期的key,达到上限之后不能进行写操作,但是仍然可以进行读操作。
18. `appendonly no`,指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no。
19. `appendfilename appendonly.aof`,指定更新日志文件名，默认为appendonly.aof。
20. `vm-enabled no`,指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中。
21. `vm-swap-file /tmp/redis.swap`, 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享。
22. `vm-max-memory 0`, 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0。
23. `vm-page-size 32`,Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不确定，就使用默认值。
24. `vm-pages 134217728`,设置swap文件中的page数量,由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的,在磁盘上每8个pages将消耗1byte的内存。
25. `vm-max-threads 4`,设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4。
26. `glueoutputbuf yes`,设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启。
27. `hash-max-zipmap-entries 64`,`  hash-max-zipmap-value 512`,指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法。
28. `activerehashing yes`,指定是否激活重置哈希，默认为开启。
29. `include /path/to/local.conf`,指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件。

## Redis数据类型
Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

##### 字符串

```
127.0.0.1:6379> set name lanyage
```
```
127.0.0.1:6379> get name
"lanyage"
```
一个键最大能存512Mb。

##### 哈希

```
127.0.0.1:6379> hmset testhash name lanyage age 18
OK
127.0.0.1:6379> hget testhash name
"lanyage"
127.0.0.1:6379> hget testhash age
"18"
```
每个 hash 可以存储 2<sup>32</sup>-1 键值对（40多亿)
##### 列表

```
127.0.0.1:6379> lpush names lanyage shufang daimengxiao
(integer) 3
```
```
127.0.0.1:6379> lrange names 0 10
1) "daimengxiao"
2) "shufang"
3) "lanyage"
```
每个列表可存储40多亿

##### 集合

```
127.0.0.1:6379> sadd host 127.0.0.1 192.168.8.131
(integer) 2
127.0.0.1:6379> sadd host helloworld
(integer) 1
```
```
127.0.0.1:6379> smembers host
1) "helloworld"
2) "127.0.0.1"
3) "192.168.8.131"
```
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。每个集合可存储40多亿个成员。
##### 有序集合

```
127.0.0.1:6379> zadd students 0 zhangsan 1 lisi 2 wangwu
(integer) 3
```
```
127.0.0.1:6379> zrange students 0 1
1) "zhangsan"
2) "lisi"
```
```
127.0.0.1:6379> zrangebyscore students 0 2
1) "zhangsan"
2) "lisi"
3) "wangwu"
```
##### 总结

|类型 | 简介 | 特性 |场景|
|:----:|:----:|:----:|:----:|
| String|二进制安全|可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M|...|
| Hash |键值对集合,即编程语言中的Map类型|适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值|存储、读取、修改用户属性|
| List |链表(双向链表)|增删快,提供了操作某一段元素的API|1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列|
| Set |哈希表实现,元素不重复|1,添加、删除,查找的复杂度都是O(1) 2,为集合提供了求交集、并集、差集等操作|1,共同好友 2,利用唯一性,统计访问网站的所有独立ip 3,好友推荐时,根据tag求交集,大于某个阈值就可以推荐|
| Sorted Set |将Set中的元素增加一个权重参数score,元素按score有序排列|数据插入集合时,已经进行天然排序|1,排行榜 2,带权重的消息队列|

## Redis键

##### Redis key 命令
|命令|描述|
|----|------|
|DEL KEY|该命令用于在 key 存在时删除 key。    |
|DUMP key |序列化给定 key ，并返回被序列化的值。|
|EXISTS key |检查给定 key 是否存在。|
|	EXPIRE key seconds| 为给定 key 设置过期时间。|
|EXPIREAT key timestamp |EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。|
|PEXPIRE key milliseconds |设置 key 的过期时间以毫秒计。|
|PEXPIREAT key milliseconds-timestamp |设置 key 过期时间的时间戳(unix timestamp) 以毫秒计|
|	KEYS pattern |查找所有符合给定模式( pattern)的 key 。|
|MOVE key db |将当前数据库的 key 移动到给定的数据库 db 当中。|
|PERSIST key |移除 key 的过期时间，key 将持久保持。|
|	PTTL key |以毫秒为单位返回 key 的剩余的过期时间。|
|TTL key|以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。|
|	RANDOMKEY |从当前数据库中随机返回一个 key 。|
|RENAME key newkey |修改 key 的名称|
|	RENAMENX key newkey |仅当 newkey 不存在时，将 key 改名为 newkey 。|
|TYPE key |返回 key 所储存的值的类型。|
##### Redis 字符串命令
|命令|描述|
|---|---|
|SET key value |设置指定 key 的值|
|	GET key |获取指定 key 的值。|
|	GETRANGE key start end |返回 key 中字符串值的子字符|
|GETRANGE key start end |返回 key 中字符串值的子字符|
|GETSET key value|将给定 key 的值设为 value ，并返回 key 的旧值(old value)。|
|GETBIT key offset|对 key 所储存的字符串值，获取指定偏移量上的位(bit)。|
|MGET key1 [key2..]|获取所有(一个或多个)给定 key 的值。|
|	SETBIT key offset value|对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。|
|SETEX key seconds value|将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。|
|SETNX key value|只有在 key 不存在时设置 key 的值。|
|SETRANGE key offset value|用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。|
|STRLEN key|返回 key 所储存的字符串值的长度。|
|MSET key value [key value ...]|同时设置一个或多个 key-value 对。|
|MSETNX key value [key value ...] |同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。|
|PSETEX key milliseconds value|这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。|
|	INCR key|将 key 中储存的数字值增一。|
|INCRBY key increment|将 key 所储存的值加上给定的增量值（increment） 。|
|INCRBYFLOAT key increment|将 key 所储存的值加上给定的浮点增量值（increment） 。|
|	DECR key|将 key 中储存的数字值减一。|
|	DECRBY key decrement|key 所储存的值减去给定的减量值（decrement） 。|
|APPEND key value|如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。|
##### Redis 哈希命令
|命令|描述|
|---|---|
|HDEL key field1 [field2] |删除一个或多个哈希表字段|
|HEXISTS key field |查看哈希表 key 中，指定的字段是否存在。|
|	HGET key field |获取存储在哈希表中指定字段的值。|
|	HGETALL key |获取在哈希表中指定 key 的所有字段和值|
|HINCRBY key field increment |为哈希表 key 中的指定字段的整数值加上增量 increment 。|
|HINCRBYFLOAT key field increment |为哈希表 key 中的指定字段的浮点数值加上增量 increment 。|
|	HKEYS key |获取所有哈希表中的字段|
|	HLEN key |获取哈希表中字段的数量|
|	HMGET key field1 [field2] |获取所有给定字段的值|
|HMSET key field1 value1 [field2 value2 ] |同时将多个 field-value (域-值)对设置到哈希表 key 中。|
|	HSET key field value  |将哈希表 key 中的字段 field 的值设为 value 。|
|	HSETNX key field value |只有在字段 field 不存在时，设置哈希表字段的值。|
|HVALS key |获取哈希表中所有值|
|HSCAN key cursor [MATCH pattern] [COUNT count] |迭代哈希表中的键值对。|
##### Redis 列表命令
|命令|描述|
|---|---|
|BLPOP key1 [key2 ] timeout |移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。|
|BRPOP key1 [key2 ] timeout |移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。|
|	BRPOPLPUSH source destination timeout |从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。|
|LINDEX key index |通过索引获取列表中的元素|
|	LINSERT key BEFORE|AFTER pivot value |在列表的元素前或者后插入元素|
|LLEN key |获取列表长度|
|	LPOP key|移出并获取列表的第一个元素|
|LPUSH key value1 [value2] |将一个或多个值插入到列表头部|
|LPUSHX key value |将一个值插入到已存在的列表头部|
|LRANGE key start stop |获取列表指定范围内的元素|
|LREM key count value |移除列表元素|
|LSET key index value |通过索引设置列表元素的值|
|LTRIM key start stop |对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。|
|	RPOP key |移除并获取列表最后一个元素|
|	RPOPLPUSH source destination |移除列表的最后一个元素，并将该元素添加到另一个列表并返回|
|	RPUSH key value1 [value2] |在列表中添加一个或多个值|
|RPUSHX key value |为已存在的列表添加值|
##### Redis 集合
|命令|描述|
|---|---|
|SADD key member1 [member2] |向集合添加一个或多个成员|
|SCARD key |获取集合的成员数|
|SDIFF key1 [key2] |返回给定所有集合的差集|
|SDIFFSTORE destination key1 [key2] |返回给定所有集合的差集并存储在 destination 中|
|SINTER key1 [key2] |返回给定所有集合的交集|
|	SINTERSTORE destination key1 [key2] |返回给定所有集合的交集并存储在 destination 中|
|SISMEMBER key member |判断 member 元素是否是集合 key 的成员|
|	SMEMBERS key |返回集合中的所有成员|
|SMOVE source destination member |将 member 元素从 source 集合移动到 destination 集合|
|	SPOP key |移除并返回集合中的一个随机元素|
|SRANDMEMBER key [count] |返回集合中一个或多个随机数|
|SREM key member1 [member2] |移除集合中一个或多个成员|
| SUNION key1 [key2] | 返回所有给定集合的并集|
|SUNIONSTORE destination key1 [key2] |所有给定集合的并集存储在 destination 集合中|
|SSCAN key cursor [MATCH pattern] [COUNT count] |迭代集合中的元素|
##### Redis 有序集合
|命令|描述|
|---|---|
|ZADD key score1 member1 [score2 member2] |向有序集合添加一个或多个成员，或者更新已存在成员的分数|
|ZCARD key |获取有序集合的成员数|
|	ZCOUNT key min max |计算在有序集合中指定区间分数的成员数|
|ZINCRBY key increment member |有序集合中对指定成员的分数加上增量 increment|
|ZINTERSTORE destination numkeys key [key ...] |计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中|
|ZLEXCOUNT key min max |在有序集合中计算指定字典区间内成员数量|
|ZRANGE key start stop [WITHSCORES] |通过索引区间返回有序集合成指定区间内的成员|
|ZRANGEBYLEX key min max [LIMIT offset count] |通过字典区间返回有序集合的成员|
|ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] |通过分数返回有序集合指定区间内的成员|
|ZRANK key member |返回有序集合中指定成员的索引|
|ZREM key member [member ...] |移除有序集合中的一个或多个成员|
|ZREMRANGEBYLEX key min max |移除有序集合中给定的字典区间的所有成员|
|	ZREMRANGEBYRANK key start stop |移除有序集合中给定的排名区间的所有成员|
|ZREMRANGEBYSCORE key min max |移除有序集合中给定的分数区间的所有成员|
|ZREVRANGE key start stop [WITHSCORES] |返回有序集中指定区间内的成员，通过索引，分数从高到底|
|ZREVRANGEBYSCORE key max min [WITHSCORES] |返回有序集中指定分数区间内的成员，分数从高到低排序|
|	ZREVRANK key member |返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序|
|ZSCORE key member |返回有序集中，成员的分数值|
|ZUNIONSTORE destination numkeys key [key ...] |计算给定的一个或多个有序集的并集，并存储在新的 key 中|
|	ZSCAN key cursor [MATCH pattern] [COUNT count] |迭代有序集合中的元素（包括元素成员和元素分值）|

##### Redis HyperLogLog
Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

|命令|描述|
|---|---|
|PFADD key element [element ...] |添加指定元素到 HyperLogLog 中。|
|	PFCOUNT key [key ...] |返回给定 HyperLogLog 的基数估算值。|
|PFMERGE destkey sourcekey [sourcekey ...] |将多个 HyperLogLog 合并为一个 HyperLogLog|

##### Redis 发布订阅
`SUBSCRIBE redisChat`用来订阅频道，`PUBLISH redisChat`用来发布消息

|命令|描述|
|---|---|
|PSUBSCRIBE pattern [pattern ...] |PUBSUB subcommand [argument [argument ...]] |
|PUBSUB subcommand [argument [argument ...]] |查看订阅与发布系统状态。|
|PUBLISH channel message |将信息发送到指定的频道。|
|PUNSUBSCRIBE [pattern [pattern ...]] |退订所有给定模式的频道。|
|SUBSCRIBE channel [channel ...] |订阅给定的一个或多个频道的信息。|
|	UNSUBSCRIBE [channel [channel ...]] |指退订给定的频道。|

##### Redis 事务
一个事务从开始到执行会经历以下三个阶段：

* 开始事务。
* 命令入队。
* 执行事务。

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。

事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

|命令|描述|
|---|---|
|DISCARD|取消事务，放弃执行事务块内的所有命令。|
|EXEC |执行所有事务块内的命令。|
|MULTI |标记一个事务块的开始。|
|	UNWATCH |取消 WATCH 命令对所有 key 的监视。|
|	WATCH key [key ...] |监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断|
##### Redis 脚本
Redis 脚本使用 Lua 解释器来执行脚本。 Redis 2.6 版本通过内嵌支持 Lua 环境。执行脚本的常用命令为 EVAL。

Eval 命令的基本语法如下：

```
redis 127.0.0.1:6379> EVAL script numkeys key [key ...] arg [arg ...]
```
以下脚本演示了redis脚本的工作过程:

	redis 127.0.0.1:6379> EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
	
	1) "key1"
	2) "key2"
	3) "first"
	4) "second"
Redis 脚本命令

|命令|描述|
|---|---|
|EVAL script numkeys key [key ...] arg [arg ...] s|执行 Lua 脚本。|	
|EVALSHA sha1 numkeys key [key ...] arg [arg ...] |执行 Lua 脚本。|
|SCRIPT EXISTS script [script ...]|查看指定的脚本是否已经被保存在缓存当中。|
|	SCRIPT FLUSH |从脚本缓存中移除所有脚本。|
|SCRIPT KILL |杀死当前正在运行的 Lua 脚本。|
|SCRIPT LOAD script |将脚本 script 添加到脚本缓存中，但并不立即执行这个脚本。|


##### Redis 连接
Redis 连接命令主要是用于连接 redis 服务。

以下实例演示了客户端如何通过密码验证连接到 redis 服务，并检测服务是否在运行：

	redis 127.0.0.1:6379> AUTH "password"
	OK
	redis 127.0.0.1:6379> PING
	PONG

|命令|描述|
|---|---|
|	AUTH password |验证密码是否正确|
|ECHO message |打印字符串|
|PING |查看服务是否运行|
|QUIT |关闭当前连接|
|	SELECT index |切换到指定的数据库|

##### Redis 服务器
```
redis 127.0.0.1:6379> INFO
```
Redis 服务器命令

|命令|描述|
|---|---|
|BGREWRITEAOF |异步执行一个 AOF（AppendOnly File） 文件重写操作|
|BGSAVE |在后台异步保存当前数据库的数据到磁盘|
|CLIENT KILL [ip:port] [ID client-id] |关闭客户端连接|
|CLIENT LIST |获取连接到服务器的客户端连接列表|
|	CLIENT GETNAME |获取连接的名称|
|CLIENT PAUSE timeout |在指定时间内终止运行来自客户端的命令|
|	CLIENT SETNAME connection-name |设置当前连接的名称|
|CLUSTER SLOTS |获取集群节点的映射数组|
|	COMMAND |获取 Redis 命令详情数组|
|COMMAND COUNT |获取 Redis 命令总数|
|	COMMAND GETKEYS |获取给定命令的所有键|
|TIME |返回当前服务器时间|
|COMMAND INFO command-name [command-name ...] |获取指定 Redis 命令描述的数组|
|CONFIG GET parameter |获取指定配置参数的值|
|CONFIG REWRITE |对启动 Redis 服务器时所指定的 redis.conf 配置文件进行改写|
|CONFIG SET parameter value |修改 redis 配置参数，无需重启|
|CONFIG RESETSTAT |重置 INFO 命令中的某些统计数据|
|DBSIZE |返回当前数据库的 key 的数量|
|	DEBUG OBJECT key |获取 key 的调试信息|
|DEBUG SEGFAULT |让 Redis 服务崩溃|
|FLUSHALL |删除所有数据库的所有key|
|FLUSHDB |删除当前数据库的所有key|
|	INFO [section] |获取 Redis 服务器的各种信息和统计数值|
|	LASTSAVE |返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示|
|MONITOR |实时打印出 Redis 服务器接收到的命令，调试用|
|ROLE |返回主从实例所属的角色|
|	SAVE |同步保存数据到硬盘|
|SHUTDOWN [NOSAVE] [SAVE] |异步保存数据到硬盘，并关闭服务器|
|SLAVEOF host port |将当前服务器转变为指定服务器的从属服务器(slave server)|
|SLOWLOG subcommand [argument] |管理 redis 的慢日志|
|	SYNC |用于复制功能(replication)的内部命令|

##### Redis 数据备份与恢复
Redis SAVE 命令用于创建当前数据库的备份。

	redis 127.0.0.1:6379> SAVE 
	OK

该命令将在 redis 安装目录中创建dump.rdb文件。

如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令，如下所示：

	redis 127.0.0.1:6379> CONFIG GET dir
	1) "dir"
	2) "/usr/local/redis/bin"	

创建 redis 备份文件也可以使用命令 BGSAVE，该命令在后台执行。

	127.0.0.1:6379> 
	Background saving started

##### Redis 安全
我们可以通过 redis 的配置文件设置密码参数，这样客户端连接到 redis 服务就需要密码验证，这样可以让你的 redis 服务更安全。	

	127.0.0.1:6379> CONFIG get requirepass
	1) "requirepass"
	2) ""

默认情况下 requirepass 参数是空的，这就意味着你无需通过密码验证就可以连接到 redis 服务。
你可以通过以下命令来修改该参数：

	127.0.0.1:6379> CONFIG set requirepass "runoob"
	OK
	127.0.0.1:6379> CONFIG get requirepass
	1) "requirepass"
	2) "runoob"
设置密码后，客户端连接 redis 服务就需要密码验证，否则无法执行命令。

AUTH 命令基本语法格式如下：	

```
127.0.0.1:6379> AUTH password
```

实例

	127.0.0.1:6379> AUTH "runoob"
	OK
	127.0.0.1:6379> SET mykey "Test value"
	OK
	127.0.0.1:6379> GET mykey
	"Test value"

##### Redis 性能测试
edis 性能测试是通过同时执行多个命令实现的。

	$ redis-benchmark -n 10000  -q
	
	PING_INLINE: 141043.72 requests per second
	PING_BULK: 142857.14 requests per second
	SET: 141442.72 requests per second
	GET: 145348.83 requests per second
	INCR: 137362.64 requests per second
	LPUSH: 145348.83 requests per second
	LPOP: 146198.83 requests per second
	SADD: 146198.83 requests per second
	SPOP: 149253.73 requests per second
	LPUSH (needed to benchmark LRANGE): 148588.42 requests per second
	LRANGE_100 (first 100 elements): 58411.21 requests per second
	LRANGE_300 (first 300 elements): 21195.42 requests per second
	LRANGE_500 (first 450 elements): 14539.11 requests per second
	LRANGE_600 (first 600 elements): 10504.20 requests per second
	MSET (10 keys): 93283.58 requests per second

以下实例我们使用了多个参数来测试 redis 性能：

	$ redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 10000 -q
	
	SET: 146198.83 requests per second
	LPUSH: 145560.41 requests per second

##### Redis 客户端连接

最大连接数

	config get maxclients
	
	1) "maxclients"
	2) "10000"

以下实例我们在服务启动时设置最大连接数为 100000：

```
redis-server --maxclients 100000
```
客户端命令

|命令|描述|
|---|---|
|CLIENT LIST|返回连接到 redis 服务的客户端列表|
|CLIENT SETNAME|设置当前连接的名称|
|CLIENT GETNAME|获取通过 CLIENT SETNAME 命令设置的服务名称|
|CLIENT PAUSE|挂起客户端连接，指定挂起的时间以毫秒计|
|CLIENT KILL|关闭客户端连接|


##### Redis 管道技术
Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。

查看 redis 管道，只需要启动 redis 实例并输入以下命令：
	$(echo -en "PING\r\n SET runoobkey redis\r\nGET runoobkey\r\nINCR visitor\r\nINCR visitor\r\nINCR visitor\r\n"; sleep 10) | nc localhost 6379
	
	+PONG
	+OK
	redis
	:1
	:2
	:3

管道技术的优势

在下面的测试中，我们将使用Redis的Ruby客户端，支持管道技术特性，测试管道技术对速度的提升效果。

	require 'rubygems' 
	require 'redis'
	def bench(descr) 
	start = Time.now 
	yield 
	puts "#{descr} #{Time.now-start} seconds" 
	end
	def without_pipelining 
	r = Redis.new 
	10000.times { 
	    r.ping 
	} 
	end
	def with_pipelining 
	r = Redis.new 
	r.pipelined { 
	    10000.times { 
	        r.ping 
	    } 
	} 
	end
	bench("without pipelining") { 
	    without_pipelining 
	} 
	bench("with pipelining") { 
	    with_pipelining 
	}

测试效果

	without pipelining 1.185238 seconds 
	with pipelining 0.250783 seconds

##### Redis分区
Redis 有两种类型分区。

最简单的分区方式是按范围分区，就是映射一定范围的对象到特定的Redis实例。

比如，ID从0到10000的用户会保存到实例R0，ID从10001到 20000的用户会保存到R1，以此类推。
这种方式是可行的，并且在实际中使用，不足就是要有一个区间范围到实例的映射表。这个表要被管理，同时还需要各 种对象的映射表，通常对Redis来说并非是好的方法。

哈希分区。
另外一种分区方法是hash分区。这对任何key都适用，也无需是object_name:这种形式，像下面描述的一样简单：

用一个hash函数将key转换为一个数字，比如使用crc32 hash函数。对key foobar执行crc32(foobar)会输出类似93024922的整数。
对这个整数取模，将其转化为0-3之间的数字，就可以将这个整数映射到4个Redis实例中的一个了。93024922 % 4 = 2，就是说key foobar应该被存到R2实例中。注意：取模操作是取除的余数，通常在多种编程语言中用%操作符实现。


##### Redis & Java

String:

	 	public static void main(String[] args) {
	        Jedis jedis = new Jedis("127.0.0.1");
	
	        jedis.set("name","shufang");
	
	        		System.out.println(jedis.get("name"));
	    }


List:

	public static void main(String[] args) {
	        Jedis jedis = new Jedis("127.0.0.1");
	
	        jedis.lpush("numbers", "1");
	        jedis.lpush("numbers", "2");
	        jedis.lpush("numbers", "3");
	
	        List<String> numbers = jedis.lrange("numbers",0,2);
	        System.out.println(numbers);
	    }
	    
keys:

	public static void main(String[] args) {
	        Jedis jedis = new Jedis("127.0.0.1");
	        jedis.lpush("numbers", "1");
	        jedis.lpush("numbers", "2");
	        jedis.lpush("numbers", "3");
	        Set<String> keys = jedis.keys("*");
	        System.out.println(keys);
	    }	  