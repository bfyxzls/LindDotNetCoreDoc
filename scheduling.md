[�ص�Ŀ¼](http://www.cnblogs.com/lori/p/7154409.html)
### ����������
1. λ��SchedulingĿ¼
2. ����JobBase,����JOB����������,��дCron���Կ����޸ĵ�������
3. ֧�ֵ���JOB,��ִ����ɺ�����ֹͣ
4. ֧�ֶ���API�ӿ�,�Ա��ȡ���޸�JOB���б��״̬
 
### Դ����չ��
�Զ���Job�ڼ̳�JobBase֮��,���������Զ�������,��������,�Ƿ񵥴�,
��ִ�й���.

```
  public abstract class JobBase : ISchedulingJob
��{
     /// <summary>
     /// ִ�мƻ�����������ִ�е�JOB֮������JOB��Ҫʵ����
     /// </summary>
     public abstract string Cron { get; }
 
     /// <summary>
     /// �Ƿ�Ϊ�������񣬺�Ϊfalse
     /// </summary>
     public virtual bool IsSingle => false;
 
     /// <summary>
     /// Job�����ƣ�Ĭ��Ϊ��ǰ����
     /// </summary>
     public virtual string JobName => GetType().Name;
 
     /// <summary>
     /// Job������ȥʵ���Լ����߼�
     /// </summary>
     protected abstract void ExcuteJob(IJobExecutionContext context);
 
     public Task Execute(IJobExecutionContext context)
     {
         try
         {
             Console.WriteLine(DateTime.Now.ToString() + "{0}���Job��ʼִ��", context.JobDetail.Key.Name);
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
### ֧����Job�Զ�װ��
��������ӵ�job,�����뵱ǰjob����,�����µ���Ŀ�����job֮��,����
���Ƶ��������ĳ����,���ĳ���ķ��ֻ��ƻ���µ�Jobװ�ص�ʱ�����
���µ����ڵ���ʱ��ȥ��ִ̬�У�

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
Demo����ƺܼ򵥣�ֻҪʵ��JobBase�����Ժͷ������ɣ������JobΪÿ����ִ��һ��
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
��������
[�ص�Ŀ¼](http://www.cnblogs.com/lori/p/7154409.html)