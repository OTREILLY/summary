
### 一、redis数据结构与对象
    
#### 1. 数据结构   
#### 简单动态字符串
```             
typedef char *sds;
struct sdshdr {
  int len;        // 记录 buf 数组中已使用字节的数量 等于 SDS 所保存字符串的长度
  int free;        // 记录 buf 数组中未使用字节的数量
  char buf[];        // 字节数组，用于保存字符串
};
``` 
```
1). '\0'结尾，但不计入字符串长度
2). O(1)读取字符串长度
3). 不会出现内存溢出
4). 空间预分配：free = len<1M ? len : 1M
5). 惰性空间释放： 通过free控制SDS字符串的缩短操作
6). 二进制安全
```       
#### 链表
```          
typedef struct listNode {
  struct listNode *prev;        // 前置节点 
  struct listNode *next;        // 后置节点   
  void *value;                // 节点的值
} listNode;
           
typedef struct list {
  listNode *head;                // 表头节点         
  listNode *tail;                // 表尾节点
  void *(*dup)(void *ptr);        // 节点值复制函数
  void (*free)(void *ptr);        // 节点值释放函数
  int (*match)(void *ptr, void *key);        // 节点值对比函数
  unsigned long len;                // 链表所包含的节点数量
} list;
```
```
1). 双端链表, 获取前驱和后置都为O(1);
2). 无环, 表头的前驱和表尾的后置都为NULL;
3). 带头指针和尾指针;
4). 带链表长度计数器;
5). 多态,  void * 保存点值,  通过list结构的dup, free, match三个属性设置节点特定函数, 可以用链表保存不同类型的值           
```            
#### 字典
```             
/*
 * 哈希表
 * 每个字典都使用两个哈希表，从而实现渐进式 rehash 。
 */
typedef struct dictht {
    dictEntry **table;    // 哈希表数组
    unsigned long size;    // 哈希表大小
    unsigned long sizemask;    // 哈希表大小掩码，用于计算索引值 等于 size - 1
    unsigned long used;    // 该哈希表已有节点的数量
} ditch;

/*哈希表节点*/
typedef struct dictEntry {
    void *key;        // 键
    union {           // 值
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    struct dictEntry *next;    // 指向下个哈希表节点，形成链表
} dictEntry;

/* 字典 */
typedef struct dict {
    dictType *type;        // 类型特定函数
    void *privdata;        // 私有数据
    dictht ht[2];         // 哈希表
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */
    int iterators;         // 目前正在运行的安全迭代器的数量
} dict;

/* 字典类型特定函数*/
typedef struct dictType {
    unsigned int (*hashFunction)(const void *key);    // 计算哈希值的函数
    void *(*keyDup)(void *privdata, const void *key);    // 复制键的函数
    void *(*valDup)(void *privdata, const void *obj);        // 复制值的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);    // 对比键的函数
    void (*keyDestructor)(void *privdata, void *key);        // 销毁键的函数
void (*valDestructor)(void *privdata, void *obj);        // 销毁值的函数
} dictType;    
```
```
1). ht数组包含两个哈希表,  一般情况下操作ht[0],  ht[1] 只有在ht[0]哈希表进行rehash时使用.  rehashidx, 记录了rehash
   的进度, 没有则为 -1 ;
2). 使用MurmurHash2算法计算哈希值
3). 解决键冲突: 链地址法,  采用头插法,  时间复杂度为O(1).
4). rehash:  (维持负载因子)
  s1. 为 ht[1] 分配空间 : 如果是扩容,  分配大小为 第一个大于等于 ht[0] .used * 2 的 2的n次幂; 如果是收缩,  分配空间
       大小为  第一个大于等于 ht[0] .used 的 2的n次幂.
  s2.  将ht[0] 上的键值对 通过重新计算键的哈希值和索引重新分配到ht[1] 上.
  s3. 这时ht[0] 将变为空表,  释放ht[0],  将ht[1] 设置为ht[0],  并在ht[1]创建一个空表, 为下次rehash做准备.

5). 渐进式rehash:
    在字典维持一个索引计数器 rehashidx, 设置为0, 表示正式工作.
    在对字典执行添加, 删除, 查找或修改时,  将 rehashidx 索引上的所有键值对 rehash 到 hd[1], 完成后 rehashidx++;
    当ht[0] 上的所有键值对被 rehash到 ht[ 1 ], 设置 rehashidx = - 1, 表示操作已经完成了.
    在此期间操作数据, 会使用ht[0] 和 ht[1]两个hash表,  例如要找某个键值, 会先在ht[0] 找, 没有就去ht[ 1 ] 找.  新增操
    作都保存到ht[ 1 ] 中. 保证了ht[0] 只减不增, 最终为空.
```

#### 跳表
```
/*跳跃表节点*/
typedef struct zskiplistNode {
  struct zskiplistNode *backward;  // 后退指针
  double score;    // 分值
  robj *obj;    // 成员对象
  struct zskiplistLevel{       // 层 
    struct zskiplistNode * forward;  // 前进指针
    unsigned int span;        // 跨度
  }level[];
  
} zskiplistNode;

/*跳跃表*/
typedef struct zskiplist {
 struct zskiplistNode *header, *tail;  // 后退指针
 unsigned long length;        // 节点的数量
 int level;        // 层数最大的节点的层数
} zskiplist;
```
```  
1). 每个跳跃表节点的层高都是1至32之间的随机数
2). 多个节点可以包含相同分值，但每个节点的成员对象必须是唯一的
3). 节点按照分值从小到大排序，当分值相同时，节点按照成员对象的大小进行排序
```   
#### 整数集合
```
     /*跳跃表*/
typedef struct intset {
 uint32_t encoding;          // 编码方式
 uint32_t length;            // 集合包含的元素数量
 int8_t contents[];         // 保存元素的数组
} intset;  
```
```
1). 一个集合只包含整数值元素，且元素数量不多时，用于实现集合对象
2). 当添加新元素到集合中时，如果新元素的类型比现有的类型要长，进行升级：
3). contents中从小到大保存数据，升级的新元素大于现有所有元素或小于现有的所有元素，添加到末尾或开头
```
#### 压缩列表
```
1). 一个列表中元素数量较少，且表中元素是小整数或短字符串
2). 由一系列特殊编码的联系内存块组成的顺序型数据结构：
    zlbytes, uint32_t, 记录整个压缩列表占用的内存字节数
    zltail, uint32_t, 记录列表表尾节点距离列表起始地址有多少字节
    zlen, uint16_t, 记录列表中节点数
    entryx, 列表节点
    lend, uint8_t, 0xFF标记压缩列表的末尾
3). 增加和删除操作，可能引发连锁更新操作

```
#### 2. 对象   
  
#### 对象的类型与编码
```        
    /* redis对象*/
typedef struct redisObject {
 unsigned type:4;        // 类型
 unsigned encoding:4;    // 编码
 void *ptr;             // 指向底层实现数据结构的指针
 int refcount;         // 引用计数
 unsigned lru:22;      // 最近访问时间
} robj;

 type: 
        REDIS_STRING        (字符串: e1,e2,e3)
        REDIS_LIST              (列表：e6, e5 )
        REDIS_HASH           (哈希: e6, e4)
        REDIS_SET              (集合: e1, e7)
        REDIS_ZSET            (有序集合: e6, e8)

 encoding:
        e1: REDIS_ENCODING_INT                    long型整数
        e2: REDIS_ENCODING_EMBSTR           embstr编码的简单动态字符串
        e3: REDIS_ENCODING_RAW                  简单动态字符串
        e4: REDIS_ENCODING_HT                     字典
        e5: REDIS_ENCODING_LINKEDLIST     双端列表
        e6: REDIS_ENCODING_ZIPLIST
        e7: REDIS_ENCODING_INTSET
        e8: REDIS_ENCODING_SKIPLIST
```
```
对象的类型检查
    命令类型: 对所有key操作的命令和对指定类型key操作命令
    类型检查：对指定类型key操作命令，执行前需要进行命令检查（redisObject-> type）

对象的多态命令
    基于类型的多态命令：DEL, TYPE,  EXPIRE, ...
    基于编码的多态命令：LLEN, ...

内存回收
    引用计数计数
    创建对象、使用对象、释放对象

对象共享
    共享值为0到9999的字符串对象

对象的空转时长
    OBJECT IDLETIME  
    volatile-lru, allkeys-lru回收算法
```

#### KEY命令：        
    * DEL
    * DUMP
    * EXISTS
    * EXPIRE
    * EXPIREAT
    * KEYS
    * MIGRATE
    * MOVE
    * OBJECT
    * PERSIST
    * PEXPIRE
    * PEXPIREAT
    * PTTL
    * RANDOMKEY
    * RENAME
    * RENAMENX
    * RESTORE
    * SORT
    * TTL
    * TYPE
    * SCAN
        字符串        
    * APPEND
    * BITCOUNT
    * BITOP
    * BITFIELD
    * DECR
    * DECRBY
    * GET
    * GETBIT
    * GETRANGE
    * GETSET
    * INCR
    * INCRBY
    * INCRBYFLOAT
    * MGET
    * MSET
    * MSETNX
    * PSETEX
    * SET
    * SETBIT
    * SETEX
    * SETNX
    * SETRANGE
    * STRLEN
        列表                  
    * BLPOP
    * BRPOP
    * BRPOPLPUSH
    * LINDEX
    * LINSERT
    * LLEN
    * LPOP
    * LPUSH
    * LPUSHX
    * LRANGE
    * LREM
    * LSET
    * LTRIM
    * RPOP
    * RPOPLPUSH
    * RPUSH
    * RPUSHX
        哈希
    * HDEL
    * HEXISTS
    * HGET
    * HGETALL
    * HINCRBY
    * HINCRBYFLOAT
    * HKEYS
    * HLEN
    * HMGET
    * HMSET
    * HSET
    * HSETNX
    * HVALS
    * HSCAN
    * HSTRLEN
        集合
    * SADD
    * SCARD
    * SDIFF
    * SDIFFSTORE
    * SINTER
    * SINTERSTORE
    * SISMEMBER
    * SMEMBERS
    * SMOVE
    * SPOP
    * SRANDMEMBER
    * SREM
    * SUNION
    * SUNIONSTORE
    * SSCAN
        有序集合 
    * ZADD
    * ZCARD
    * ZCOUNT
    * ZINCRBY
    * ZRANGE
    * ZRANGEBYSCORE
    * ZRANK
    * ZREM
    * ZREMRANGEBYRANK
    * ZREMRANGEBYSCORE
    * ZREVRANGE
    * ZREVRANGEBYSCORE
    * ZREVRANK
    * ZSCORE
    * ZUNIONSTORE
    * ZINTERSTORE
    * ZSCAN
    * ZRANGEBYLEX
    * ZLEXCOUNT
    * ZREMRANGEBYLEX


### 二、redis数据库
  
#### 1. 数据库
```
struct redisServer{
    redisDb *db;  // 数据库数组  
    int dbnum;  // 默认16个
}redisServer;

struct redisClient{
    redisDb *db;  // 数据库数组  
    int dbnum;  // 默认16个
}redisClient;
```

SELECT n   切换数据库

#### 2. 数据库键空间
```
typedef  struct redisDb{
    dict *dict;      // 数据库键空间，保存所有的键值对
    dict *expires;  // 过期字典，保存所有键的过期时间
}
```   
```
键空间中的每个键都是字符串对象，对应的值是任意Redis对象
添加新键、删除键、更新键、读取键值

键对象的过期时间：
   键的生存时间或过期时间： 过期字典（键对象：UNIX时间戳（单位ms））            
   设置过期时间：
        EXPIRE <key> <ttl>
        PEXPIRE <key> <ttl>
        EXPIREAT <key> <timestamp>
        PEXPIREAT <key> <timestamp>
   移除过期时间：
        PERSIST <key>
   计算剩余时间：
        TTL <key>    
        PTTL <key>  
   过期键删除策略：
        定时删除、惰性删除、定时删除
   AOF/RDB持久化和复制对过期键的处理：
        
```

### 3. 持久化
```
RDB持久化
    1）将某个时间点上的数据库状态保存到一个RDB文件中，RDB文件是一个经过压缩的二进制文件。
    2）SAVE/BGSAVE:
            SAVE命令会阻塞Redis服务器进程，直到RDB文件创建完毕，期间，服务器不能处理任何命令请求。
            BGSAVE命令不会阻塞Redis服务器进程，而是派生出一个子进程负责创建RDB文件，服务器进程处理请求
    3）自动载入RDB文件（服务器启动时，自动执行）
    4）BGSAVE执行期间，SAVE和BGSAVE命令会被拒绝
    5）SAVE/BGSAVE执行策略：
            手动执行或自动间隔性执行
    6）RDB文件：
 AOF持久化
    1）保存服务器所执行的写命令来记录数据库的状态，被写入AOF文件的所有命令都是以redis的命令请求协议格式保
         存的。
    2）AOF持久化步骤：
         命令追加、文件写入、文件同步
         事件循环/aof_buf
   3）flushAppendOnlyFile策略：    
            always:   将oaf_buf缓冲区中的所有内容写入并同步到AOF文件
            everysec: 将oaf_buf缓冲区中的所有内容写入到AOF文件，定时同步
            no: 将oaf_buf缓冲区中的所有内容写入到AOF文件，但不同步，积累一段时间后同步
    4）AOF文件载入与数据还原
            伪客户端执行AOF中的写命令
    5）AOF重写
    6）BGREWRITEAOF与SAVE/BGSAVE
    7）AOF与RDB：
            优先载入AOF文件
```

#### 4. 事件
```
1）Redis服务器是一个事件驱动程序： 文件事件、时间事件
2）文件事件：
    文件事件处理器：基于Reactor模式实现的网络通信程序
    文件事件是对套接字操作的抽象：AE_READABLE\AE_WRITABLE
3）时间事件：
    定时事件和周期性事件
4）事件调度和执行规则：
```    

#### 5. 客户端与服务器

### 三、Redis集群
    
1. 复制 </br>

2. 集群


### 四、独立功能

```
redis事务：MULTI EXEC DISCARD WATCH UNWATCH
redis事务：事务队列，入队与出队
redis事务：ACID
redis事务：不支持回滚 （1.  2.简单模型）
redis事务： WATCH 与 乐观锁
```


### 参考
《Redis设计与实现》 黄健宏 著  </br>
https://github.com/menwengit/redis_source_annotation/blob/master/README.md </br>
https://zhuanlan.zhihu.com/p/21368183?refer=zhangtielei   </br>
https://www.cnblogs.com/chenpingzhao/archive/2017/06/10/6965164.html  </br>
