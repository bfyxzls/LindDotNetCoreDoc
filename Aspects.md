### Aspect面向方面编程
面向侧面的程序设计（aspect-oriented programming，AOP，又译作面向方面的程序设计、观点导向编程、剖面导向程序设计）是计算机科学中的一个术语，指一种程序设计范型。该范型以一种称为侧面（aspect，又译作方面）的语言构造为基础，侧面是一种新的模块化机制，用来描述分散在对象、类或函数中的横切关注点（crosscutting concern）。
侧面的概念源于对面向对象的程序设计的改进，但并不只限于此，它还可以用来改进传统的函数。与侧面相关的编程概念还包括元对象协议、主题（subject）、混入（mixin）和委托。
1. 术语
1. 拦截器
1. 日志拦截器
1. 缓存拦截器
#### 术语
1. 关注点（concern）：对软件工程有意义的小的、可管理的、可描述的软件组成部分，一个关注点通常只同一个特定概念或目标相关联。
1. 主关注点（core concern）：一个软件最主要的关注点。
1. 关注点分离（separation of concerns，SOC）：标识、封装和操纵只与特定概念、目标相关联的软件组成部分的能力，即标识、封装和操纵关注点的能力。
1. 方法（method）：用来描述、设计、实现一个给定关注点的软件构造单位。
1. 横切（crosscut）：两个关注点相互横切，如果实现它们的方法存在交集。
1. 支配性分解（dominant decomposition）：将软件分解成模块的主要方式。传统的程序设计语言是以一种线性的文本来描述软件的，只采用一种方式（比如：类）将软件分解成模块；这导致某些关注点比较好的被捕捉，容易进一步组合、扩展；但还有一些关注点没有被捕捉，弥散在整个软件内部。支配性分解一般是按主关注点进行模块分解的。
1. 横切关注点（crosscutting concerns）：在传统的程序设计语言中，除了主关注点可以被支配性分解方式捕捉以外，还有许多没有被支配性分解方式捕捉到的关注点，这些关注点的实现会弥散在整个软件内部，这时这些关注点同主关注点是横切的。
1. 侧面（aspect）：在支配性分解的基础上，提供的一种辅助的模块化机制，这种新的模块化机制可以捕捉横切关注点。
#### 拦截器
在方法执行前或者执行后注入新的代码段，让新的代码段的功能注入到原方法里，其中原方法需要是接口方法或者虚方法，原因是我们要重写原方法！
#### 日志拦截器
这种拦截器比较简单，只在方法执行前，后进行日志行为的注入，不需要返回结果．
#### 缓存拦截器
这种拦截器比起日志拦截器来说就复杂一些，它需要通过缓存的key来返回结果，同时集成了缓存的逻辑及缓存持久化组件等．
##### LindAspect缓存的介绍
* 存储　：数据持久化到redis中间件里
* 结构　：采用redis-hashset进行存储
* 键前缀：Lind_Caching_
* 键组成：前缀＋方法名
* hashkey：方法参数的组合
* hashval:存储方法返回的结果，以json格式存储
##### 代码Demo
```C#
/// <summary>
/// 为方法添加缓存行为
/// </summary>
/// <returns></returns>
[CachingAspect]
public virtual List<People> GetHello()
{
    return new List<People>
    {
        new People("test1","ok",DateTime.Now),
        new People("test2","sad",DateTime.Now),
        new People("test2","force",DateTime.Now),
    };
}
[Fact]
public void FuncInvoke()
{
    var fun = ProxyFactory.CreateProxy<AspectTest, AspectTest>();
    var obj1 = fun.GetHello();
    Thread.Sleep(1000);
    var obj2 = fun.GetHello();
    Assert.Equal(obj1, obj2);
}
```