

[TOC]

###### 使用场景

项目当中使用了SSDB作为缓存媒介。但在可预见的将来，服务的请求量可能飞速上涨，从500QPS飞升至5000QPS乃至10000QPS。考虑到SSDB的性能可能无法满足要求，缓存需要 能够同时支持REDIS。



SSDB客户端虽然兼容REDIS协议，但还是存在微小的差异。譬如

| 存储  | 删除队列 | 删除哈希 | 删除有序集 |
| ----- | -------- | -------- | ---------- |
| SSDB  | qclear   | hclear   | zclear     |
| REDIS | del      | del      | del        |

无论是SSDB或REDIS，在业务方看来他们都是缓存。如果缓存介质的切换要求业务方作出代码变更的话，其修改与测试的代价都不小。那么，如何设计代码来一次性满足以上需求呢？



###### 解决方案

1. 定义接口

   ```go
   type CacheType 			int
   
   const (
   	RedisCacheType 		CacheType = iota + 1
   	SSDBCacheType
   )
   
   type Cache interface {
   	CacheType() 			CacheType
   
   	DelHash(key string) 	error
   	DelZSet(key string) 	error
   	DelQueue(key string) 	error
   }
   ```

2. 实现接口

   ```go
   type RedisCache struct{
   	realClient 	*redis.Client
   }
   
   type SSDBCache struct{
   	realClient 	*ssdb.Client
   }
   
   ```

3. 实现外观模式

   ```go
   var Cache Cacher
   func init()  {
   	Cache = newCache(SSDBCacheType)
   }
   
   func newRedisCache() *RedisCache {
   	return &RedisCache{
   		realClient: redis.NewClient(
   			&redis.Options{
   				Network:  		"TCP",
   				Addr: 	 		"127.0.0.1",
   			}),
   	}
   }
   
   func newSSDBCache() *SSDBCache {
   	client, err := ssdb.Connect("127.0.0.1", 8888)
   	if err != nil {
   		return nil
   	}
   
   	return &SSDBCache{
   		realClient: client,
   	}
   }
   ```

###### 模式优点

- 对外屏蔽了实现细节，使外部调用简单化了。
- 对外屏蔽了实现细节，同时还使得程序具有较好的扩展性。