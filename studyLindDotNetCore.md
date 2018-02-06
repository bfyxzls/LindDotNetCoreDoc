[大叔博客](http://www.cnblogs.com/lori)
## LindDotNetCore基础介绍
1. 运行环境
2. 配置文件
3. 服务的注册
4. 配置文件的注册
5. 服务的使用
6. 配置文件的使用
#### 运行环境
vs2017+.netcore2.0，vs需要升级到最新包
#### 配置文件
appsetting.json，我们提出了开发环境，测试环境和生产环境，分别对应不同的文件
  * 开发：Development,appsetting.Development.json
  * 测试：Staging,appsetting.Development.json
  * 生产：Producting,appsetting.Development.json
#### 服务的注册
在.net core里，包括在LindDotNetCore里，服务的注册是在startup里进行，你可以方便的控制每个组件的生命周期。
  * 单例，整个进程使用同一个实例，像redis,mongodb,日志
  * 线程单例，在一个线程里它是唯一的实例，在api环境下，你的一个http请求下来，一个对象只生产一次，像http请求链
  * 瞬息，每次注入时，都会生产一个新的实体。像仓储对象，数据上下文
```c#
public void ConfigureServices(IServiceCollection services)
{

    //Lind.DotNetCore封装的一些模块
    services.AddLog4Logger(o =>
    {
        o.Log4ConfigFileName = "log4.config";
        o.ProjectName = "test";
    });
    services.UseDapper(o =>
    {
        o.ConnString = $"Data Source={Directory.GetCurrentDirectory()}/intergratetest.db";
        o.DbType = Lind.DotNet
	}
}
```
#### 配置文件的注册
> 大叔封装了配置文件的注入和获取方法，注入需要依赖环境变量，它在startup初始时被生产。
```c#
public Startup(IConfiguration configuration, IHostingEnvironment env)
{
    ConfigFileHelper.Set(env: env);
    Configuration = configuration;
}
```
#### 服务的使用
我们的服务在startup里一次性被注入，然后在每个控制器的构造方法里被使用，注意：**我们的服务支持依赖型注入**，
这点对我们重要，比如一个服务的生产依赖于另一个服务，那么，这种关系由core DI帮我们实现！
```c#
[Route("api/[controller]")]
public class ValuesController : Controller
{
 ILogger _logger;
 public ValuesController(ILogger logger)
 {
    _logger = logger;
 }
```
#### 配置文件的使用
我们可以直接使用Utils命名空间下的ConfigFileHelper对象，它里面有Get方法，用来获取具体的配置节点
注意，咱们的配置节点支持强类型和字符串两种，**强类型**要求你提供泛型，字符串只要输入名称就可以
返回具体的值了。
```c#
var options = new EFConfig();
//装饰
configure?.Invoke(options);
//优先级控制
ObjectMapper.MapperTo(options, ConfigFileHelper.Get<EFConfig>());
```
待续...
[回到目录](http://www.cnblogs.com/lori/p/7154409.html)