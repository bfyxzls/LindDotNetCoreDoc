[�ص�Ŀ¼](http://www.cnblogs.com/lori/p/7154409.html)
### ְ����ģʽ���
1. �������
1. �����
1. ��������
1. ����������
  * ����ֻ������֯�������̵Ĵ��򣬶�����ʵ��ϸ��û��Ȥ
  * ��������ֻʵ���Լ���ע�Ĵ��룬����һ������δ֪
### �ھ�������е�����
��������
```c#
 public interface ICommand
 {
     void Execute(CommandParameters parameters);
 }
 ```
��������
```c#
 /// <summary>
 /// ����������������
 /// </summary>
 public abstract class WorkFlow
 {
     protected WorkFlow Next; // �����̶���
     protected object SharedObj;  // �������
 
     // ���ú����
     public WorkFlow SetNext(WorkFlow next)
     {
         Next = next;
         return next;
     }
 
     // ������������
     public virtual void ProcessRequest(CommandParameters command)
     {
         if (Next != null)
             Next.ProcessRequest(command);
     }
 }
```
������Ҫ�Ѳ������ݸ�ÿ����������
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
���濴һ��ְ����ģʽ��ľ�������;������̣����裩��ÿ�����������������һ����ʲô
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
��������
[�ص�Ŀ¼](http://www.cnblogs.com/lori/p/7154409.html)