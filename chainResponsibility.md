[回到目录](http://www.cnblogs.com/lori/p/7154409.html)
### 职责链模式组成
1. 三大对象
1. 命令处理
1. 处理流程
1. 命令上下文
  * 命令只负责组织各个流程的次序，对流程实现细节没兴趣
  * 具体流程只实现自己关注的代码，对下一个流程未知
### 在具体代码中的体现
抽象命令
```c#
 public interface ICommand
 {
     void Execute(CommandParameters parameters);
 }
 ```
抽象流程
```c#
 /// <summary>
 /// 工作流－抽象处理者
 /// </summary>
 public abstract class WorkFlow
 {
     protected WorkFlow Next; // 定义后继对象
     protected object SharedObj;  // 共享对象
 
     // 设置后继者
     public WorkFlow SetNext(WorkFlow next)
     {
         Next = next;
         return next;
     }
 
     // 抽象请求处理方法
     public virtual void ProcessRequest(CommandParameters command)
     {
         if (Next != null)
             Next.ProcessRequest(command);
     }
 }
```
命令需要把参数传递给每个工作流程
```c#
  public class CommandParameters
  {
      public string CommandType { get; set; }
      public string JsonObj { get; set; }

      public CommandParameters(string type, string jsonObj)
      {
          CommandType = type;
          JsonObj = jsonObj;
      }
  }
```
下面看一个职责链模式里的具体命令和具体流程（步骤），每个步骤可以设置它下一步是什么
```c#
  public class CommandInsert : ICommand
  {

      public void Execute(CommandParameters parameters)
      {
          WorkFlow workFlow = new WorkFlow_InsertLogger();
          workFlow.SetNext(new WorkFlow_InsertAudit());
          workFlow.ProcessRequest(parameters);
      }
  }

  public class CommandUpdate : ICommand
  {
      public void Execute(CommandParameters parameters)
      {
          WorkFlow workFlow = new WorkFlow_InsertAudit();
          workFlow.SetNext(new WorkFlow_InsertLogger());
          workFlow.ProcessRequest(parameters);
      }
  }

  public class WorkFlow_InsertLogger : WorkFlow
  {
      public override void ProcessRequest(CommandParameters command)
      {
          System.Console.WriteLine("WorkFlow1");
          ProcessRequest(command);
      }
  }
  public class WorkFlow_InsertAudit : WorkFlow
  {
      public override void ProcessRequest(CommandParameters command)
      {
          System.Console.WriteLine("WorkFlow2");
          ProcessRequest(command);
      }
  }

  public class ChainResponsibility
  {
      [Fact]
      public void Test1()
      {
          var command = new CommandInsert();
          command.Execute(new CommandParameters("test", "OK"));
      }

      [Fact]
      public void Test2()
      {
          var command = new CommandUpdate();
          command.Execute(new CommandParameters("test", "OK"));
      }
  }
```
待续……
[回到目录](http://www.cnblogs.com/lori/p/7154409.html)