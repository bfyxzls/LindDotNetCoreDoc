[大叔博客](http://www.cnblogs.com/lori)
### LindDotNetCore相关模块介绍
1. [x] 全局都是依赖DI
1. [x] 消息队列
1. [x] NoSql
1. [x] Caching
1. [x] 仓储
1. [x] 服务总线
1. [x] Solr
1. [x] 调度
1. [x] 日志
1. [x] Asspect拦截组件
1. [ ] UAA授权 
1. [ ] 各种组件环境的搭建
1. [x] 各模块单元测试编写
#### DI统一战线
LindDotNet框架同样采用了全局DI注入的方式来使用模块对象的，这种松耦合的设计对于单元测试
是很方便人。
```c#
services.AddLog4Logger(o =>
{
    o.Log4ConfigFileName = "log4.config";
    o.ProjectName = "test";
});
services.UseDapper(o =>
{
    o.ConnString = $"Data Source=/Data/intergratetest.db";
    o.DbType = Lind.DotNetCore.Repository.DbType.SqlLite;
});
```
#### 消息队列
消息队列主要使用'rabbitmq,kafka'实现的，用来解耦项目，处理高并发任务和耗时任务，生产者
不需要关心是谁来消费，它只管把消息发到队列中；而消费者不关心消息如何产生，只把消费按着
业务逻辑去处理掉！
```c#
services.AddRabbitMQ(o =>
{
    o.ExchangeName = "Piliapa.zzl";
    o.MqServerHost = "192.168.200.214";
    o.VirtualHost = "/";
    o.ExchangeType = "topic";
});
```
#### NoSql
目前框架的NoSql部分由`redis和mongodb`组成，之所有选择这两种框架最大的原因就是它们覆盖了
NoSql所有的使用场景，像redis用来存储k/v键值对，支持5大数据结构；而mongodb用来存储文档
型数据，支持复杂的查询，嵌套查询等。
```c#
services.AddRedis(o =>
{
    o.Host = "localhost:6379";
    o.AuthPassword = "";
    o.IsSentinel = 1;
    o.ServiceName = "mymaster";
    o.Proxy = 0;
});
```
#### Caching
数据缓存是比较重要的部分，用来存储一些热数据，目前分布式环境使用redis，单机可以直接使用
运行时缓存。
```c#
services.AddRuntimeCache(o =>
{
    o.CacheKey = "lindCache";
    o.ExpireMinutes = 5;
});
```
#### 仓储
仓储主要简化数据持久化的操作，对外提供简单的CURD操作接口，使用者直接调用即可，不需要干预SQL语句，
从这点上来说，开发效率确实提升了不少。目前大叔框架里集成了`ef,dapper,mongodb,redis,elastic`等仓储，其中
EF和Dapper可以操作sqlserver,mysql,sqllite等数据库。
```c#
services.UseDapper(o =>
{
    o.ConnString = $"Data Source={Directory.GetCurrentDirectory()}/intergratetest.db";
    o.DbType = Lind.DotNetCore.Repository.DbType.SqlLite;
});
```
#### 服务总线
服务总线主要是用来解耦项目的层与层之间的调用，让程序员把关注点放在业务上，目前框架提供了IOC模式的事件，
基于简单内存字典存储的事件等。
```c#
services.AddIocBus();
services.AddInMemoryBus();
```
#### Solr
Solr是在`Lucene`基础之前开发的，使用java编写，一般部署在tomcat上，有自己的图像管理界面，可以用来管理core，
一般地，我们在设计一个core时，需要为它建立对应的实体，与它的core里的属性对应起来；solr有丰富的插件，像一些
中文分词包，索引包等。
```c#
services.AddSolrNet(o =>
{
    o.ServerUrl = "http://192.168.200.214:8081/solr/system_companysubject";
    o.UserName = "sa";
    o.Password = "sa";
});
```
#### 调度服务
调度服务是以`quartz`为核心，并对它的功能进行了封装，支持实时添加的任务，这一点使用了windows/linux的目录监控事件
，也是.netcore帮我们实现的，我们只需要订阅相关事件即可。
```c#
var watcher = new FileSystemWatcher
{
    Path = AppDomain.CurrentDomain.BaseDirectory,
    NotifyFilter = NotifyFilters.Attributes |
                   NotifyFilters.CreationTime |
                   NotifyFilters.DirectoryName |
                   NotifyFilters.FileName |
                   NotifyFilters.LastAccess |
                   NotifyFilters.LastWrite |
                   NotifyFilters.Security |
                   NotifyFilters.Size,
    Filter = "*.dll"
};
// quartz运行时，可以添加新job，但不能覆盖，删除等
watcher.Created += new FileSystemEventHandler((o, e) =>
{
    foreach (var module in Assembly.LoadFile(e.FullPath).GetModules())
    {
        foreach (var type in module.GetTypes().Where(i => typeof(ISchedulingJob).IsAssignableFrom(i)))
        {
            JoinToQuartz(type, DateTimeOffset.Now);
        }
    }
});
//Start monitoring.
watcher.EnableRaisingEvents = true;
```
#### 日志
日志框架与之前的Lind框架里日志差别不大，只是把对象的生命周期移到了DI容器去统一管理，都采用单例方式，目前日志框架提供了
对log4net的支持，同时轻量级日志可以使用lindlogger来实现。
```c#
services.AddLog4Logger(o =>
{
    o.Log4ConfigFileName = "log4.config";
    o.ProjectName = "test";
});
```
#### Asspect拦截组件
方法拦截在微软`mvc,api`框架里应用十分广泛，可以在方法执行前与执行后动态添加一切逻辑，而不需要关注方法细节，实现拦截行为
的开发人员不需要去关注方法细节，这利用了面向对象的封装特性，而也符合开闭原则，因为你可以在不修改原来代码的情况下，动态
为它添加行为。
```c#
[Fact]
public void FuncInvoke()
{
    var obj = ProxyFactory.CreateProxy<AspectTest, AspectTest>();
    Assert.Equal("OK", obj.GetHello());
}
[Fact]
public void ActionInvoke()
{
    var obj = ProxyFactory.CreateProxy<AspectTest, AspectTest>();
    obj.SetHello();
    Assert.Equal(1, 1);
}
```
待续...
[回到目录](http://www.cnblogs.com/lori/p/7154409.html)