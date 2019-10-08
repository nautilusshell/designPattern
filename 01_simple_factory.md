

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
   }
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
   
   func NewLogger(loggerType int) CommLogger {
   	if loggerType == DBLoggerType {
   		return &DbLogger{}
   	} else if loggerType == FileLoggerType {
   		return &FileLogger{}
   	} 
   		
   	return nil
   }
   ```

###### 模式优点

- 简化了组件的使用方式。调用方始终通过相同的接口来实现业务需求，而不需关注接口内部的实现细节。
- 使业务调用与组件的实现细节解耦开来。调用方只修改简单的参数值，即可实现接口的切换而不需要修改其他任何业务代码。