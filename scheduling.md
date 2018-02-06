[回到目录](http://www.cnblogs.com/lori/p/7154409.html)
### 任务调度组件
1. 位于Scheduling目录
2. 基类JobBase,所有JOB都派生自它,重写Cron属性可以修改调度周期
3. 支持单次JOB,即执行完成后马上停止
4. 支持对外API接口,以便获取和修改JOB的列表的状态
 
### 源代码展现
自定义Job在继承JobBase之后,可以设置自动的名称,调度周期,是否单次,
和执行过程.

```
  public abstract class JobBase : ISchedulingJob
　{
     /// <summary>
     /// 执行计划，除了立即执行的JOB之后，其它JOB需要实现它
     /// </summary>
     public abstract string Cron { get; }
 
     /// <summary>
     /// 是否为单次任务，黑为false
     /// </summary>
     public virtual bool IsSingle => false;
 
     /// <summary>
     /// Job的名称，默认为当前类名
     /// </summary>
     public virtual string JobName => GetType().Name;
 
     /// <summary>
     /// Job具体类去实现自己的逻辑
     /// </summary>
     protected abstract void ExcuteJob(IJobExecutionContext context);
 
     public Task Execute(IJobExecutionContext context)
     {
         try
         {
             Console.WriteLine(DateTime.Now.ToString() + "{0}这个Job开始执行", context.JobDetail.Key.Name);
             ExcuteJob(context);
         }
         catch (Exception ex)
         {
             Console.WriteLine(this.GetType().Name + "error:" + ex.Message);
         }
 
         return Task.CompletedTask;
     }
 
 }

```
### 支持新Job自动装载
我们新添加的job,可以与当前job并行,你在新的项目中添加job之后,把它
复制到调度中心程序后,中心程序的发现机制会把新地Job装载到时队列里，
在新的周期到来时候，去动态执行！

```
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
```
Demo的设计很简单，只要实现JobBase的属性和方法即可，下面的Job为每分钟执行一次
```
 public class HelloJob : JobBase
 {
     public override string Cron => "0 0/1 * * * ?";
     protected override void ExcuteJob(IJobExecutionContext context)
     {
         Console.WriteLine("hello");
     }
 }
```
待续……
[回到目录](http://www.cnblogs.com/lori/p/7154409.html)