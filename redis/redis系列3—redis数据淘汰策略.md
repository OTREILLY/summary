### redis数据淘汰策略
```
volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用 的数据淘汰
volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数 据淘汰
volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据 淘汰
allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
no-enviction（驱逐）：禁止驱逐数据
```
默认的内存策略是noeviction，在Redis中LRU算法是一个近似算法，默认情况下，Redis随机挑选5个键，并且从中选取一个最近最久未使用的key进行淘汰，在配置文件中可以通过maxmemory-samples的值来设置redis需要检查key的个数,但是栓查的越多，耗费的时间也就越久,但是结构越精确(也就是Redis从内存中淘汰的对象未使用的时间也就越久~),设置多少，综合权衡。 </br>

LRU 数据淘汰机制 </br>
　　在服务器配置中保存了 lru 计数器 server.lrulock，会定时（redis 定时程序 serverCorn()）更新，server.lrulock 的值是根据 server.unixtime 计算出来的。 从 struct redisObject 中可以发现，每一个 redis 对象都会设置相应的 lru。可以想象的是，每一次访问数据的时候，会更新 redisObject.lru。 </br>

LRU 数据淘汰机制是这样的：　　</br>
在数据集中随机挑选几个键值对，取出其中 lru 最小的键值对淘汰。所以，你会发现，redis并不是保证取得所有数据集中最近最少使用（LRU）的键值对，而只是随机挑选的几个键值对中的。  </br>
