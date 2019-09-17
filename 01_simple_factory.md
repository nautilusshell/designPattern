

[TOC]

###### 使用场景

在一个拥有多个模块的项目当中，需要在公共模块中提供一个日志组件供业务模块使用。这个组件必须满足以下条件两个条件：

1. 至少支持DB和文件两种存储媒介（当然，DB的表格字段是预定义的）

2. 支持存储类型配置化

   

请问，如何实现以上需求呢？



###### 解决方案

1. 定义接口

   ```go
   type CommLogger interface {   
       Log(format string, args ...interface{})
   }d
   ```

2. 实现接口

   ```go
   type DbLogger struct{}
   type FileLogger struct{}
   ```

3. 实现轻工厂

   ```go
   const (   
       DBLoggerType   int = iota + 1   
       FileLoggerType
   )
   
   func CreateLogger(loggerType int) CommLogger {
   	if loggerType == DBLoggerType {
   		return &DbLogger{}
   	} else if loggerType == FileLoggerType {
   		return &FileLogger{}
   	} 
   		
   	return nil
   }
   ```