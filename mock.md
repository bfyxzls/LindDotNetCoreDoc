[回到目录](http://www.cnblogs.com/lori/p/7154409.html)
### Mock在单元测试里的意义
mock测试就是在测试过程中，对于某些不容易构造或者不容易获取的对象，用一个虚拟的对象来创建以
便测试的测试方法。**一个闹钟**根据时间来进行提醒服务，如果过了**下午5点钟**就播放音频文件
提醒大家下班了，如果我们要利用真实的对象来测试的话就只能苦苦等到下午五点，然后把耳朵放在音箱
旁，我们应该利用mock对象[1]  来进行测试，这样我们就可以模拟控制时间了，而不用苦苦等待时钟转
到下午5点钟了。
#### 为什么要用Mock
1. 模拟接口的方法实现，方便测试，不需要额外建立新的类型
2. 对集成测试很有必要
3. 体现了面向接口编程的重要性和必要性
3. 一般将数据层进行Mock，通过对数据的模拟，来实现业务的准确性
#### 输入参数和预期结果
我们可以定义两个对象，输入参数是我们给测试方法传递的原始数据，它通过计算逻辑生产新的结果；
而预期结果是我们从真实环境中通过输入参数产生的正式结果；在经过mock测试之后，我们把真实的预
期结果和测试产生的结果进行对比，这样可以验证业务逻辑的正确性！
#### 使用方法
```c#
 //注册一个mock对象，并重写它的方法GetClosing，伪造它的返回结果
 _report_CashFlowDao = new Mock<IReport_CashFlowDao>();
 _report_CashFlowDao.Setup(p => p.GetClosing(270, new DateTime(2017, 10, 31))).Returns(() =>
 {
     return _sheetReportList;
 });
```
下面业务层方法依赖于它，通过构造方法把它注入进来
```c#
 _cashFlowService = new CashFlowService(_report_CashFlowDao.Object);
 _cashFlowService.HandleOrder(1139);
```
通过上面代码我们完成的一个业务场景的mock过程，并最终调用了它的HandlerOrder方法，在这里我们与
数据库交互的IReport_CashFlowDao对象是被模拟出来的，我们可以为它提供多份模拟数据，以便更客观
的测试结果的正确性！
待续……
[回到目录](http://www.cnblogs.com/lori/p/7154409.html)