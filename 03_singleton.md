

[TOC]

###### 使用场景

在进程的生命周期中，只被初始化一次



###### 解决方案

1. 定义原子操作变量

   ```go
   var (
   	instance 	Config
   	once    	sync.Once
   )
   
   type Config struct {
   }
   ```
   
2. 实现单例

   ```go
   func NewConfig() *Config {
   	once.Do(func() {
   		instance = &Config{}
   	})
   
   	return instance
   }
   
   ```

