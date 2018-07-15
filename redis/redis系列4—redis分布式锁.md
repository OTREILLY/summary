### redis分布式锁

#### 1. 分布式锁



#### 2. redis分布式锁实现

```
package com.dev.wuxl.distributed_locks;
import redis.clients.jedis.Jedis;
import java.util.ArrayList;
import java.util.List;

/**
 * @author <a href="mailto:wu.xuanle@immomo.com">wu.xuanle</a>
 * @create 18/5/31
 */
public class MyLock {

  public boolean tryGetLock(Jedis jedis, String lockKey, String requestid, int expireTime){
    String res = jedis.set(lockKey, requestid, "NX", "PX", expireTime);
    return "OK".equals(res);
  }

  public boolean releaseLock(Jedis jedis, String lockKey, String requestid){
    String script =
      "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
    List<String> keys = new ArrayList<String>();
    keys.add(lockKey);
    List<String> values = new ArrayList<String>();
    keys.add(requestid);
    Object result = jedis.eval(script, keys, values);
    return "OK".equals(result);
  }
}
```
