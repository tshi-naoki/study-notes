转载：https://www.cnblogs.com/willick/p/14078259.html

先是半个月前 @碧水青荷 童鞋的一句话“大家都说不要随便 Task.Run(()=>{}) 这样写”，当时没有想太多，这句话并没有引起我注意，只顾着回答他“不想在代码中加 async/await 该怎么做”的问题。

然后这句话被 @裤兜 童鞋注意到，昨天问了我为什么。我当时也很纳闷，Task.Run 在并行场景中很常见啊，为什么大家会有不要随便使用的说法。很遗憾，我当时脑海里认为这种说法只是空穴来风，并没有细究。

我有个习惯，就是下班路上在地铁上快速复盘一下今天发生的事情。当时这个问题刚好就在脑海里闪现了一下，“为什么大家都说不要随便使用 Task.Run”。突然想起了多年前的一个晚上……哦，难道是“Ta”？

对，应该就是它，内存泄露，除了这个原因我再也想不到其它原因了。因为我隐约记得多年前我确实踩过一次这个坑，也可能是两次。

没错，Task.Run 使用不当，一不留意就会有内存泄露的问题。

我们先来看一段代码：

```c#
public class MyClass
{
    private int _id;
    private Logger<MyClass> _logger;
    
    public MyClass(Logger<MyClass> logger)
    {
        _logger = logger;
    }
 
 
    public Task Foo(Logger<MyClass> logger)
    {
        return Task.Run(() =>
        {
            _logger.LogInformation($"Executing job with ID {_id}");
            // do sth.
        });
    }
}
```

在这段代码中，私有成员 `_id` 被 `Task.Run` 的匿名方法捕获使用，进而导致 `MyClass` 实例被引用。当外部使用完 `MyClass` 实例时，本该由 `GC` 回收的时候却发现它还被其它资源引用着，所以 `GC` 认为该实例不应用被回收，也就可能永远失去了被回收的机会。

道理很简单，我就不再用示例演示了。解决办法也很简单，想必很多人都知道，就是使用本地变量。

```c#
public class MyClass
{
    private int _id;
    private Logger<MyClass> _logger;
    
    public MyClass(Logger<MyClass> logger)
    {
        _logger = logger;
    }
 
 
    public Task Foo(Logger<MyClass> logger)
    {
        var localId = _id;
        return Task.Run(() =>
        {
            _logger.LogInformation($"Executing job with ID {localId}");
            // do sth.
        });
    }
}
```

通过将值分配给一个本地变量，类就没有成员被捕获，即避免了潜在的内存泄漏。

内存泄漏问题在 `Task.Run` 身上发生很常见，容易被大家记住，容易提高警觉。其实不光是 `Task.Run`，其它地方使用了匿名方法也同样要小心，比如这个示例：

```c#
public class MyClass
{
    private int _id;
    private Logger<MyClass> _logger;
    private JobQueue _jobQueue;
 
    public MyClass(Logger<MyClass> logger, JobQueue jobQueue)
    {
        _logger = logger;
        _jobQueue = jobQueue;
    }
 
    public void Foo()
    {
        _jobQueue.EnqueueJob(() =>
        {
             _logger.LogInformation($"Executing job with ID {_id}");
            // do sth.
        });
    }
}
```

也有内存泄漏的问题。

总之，<font color="tomato">任何使用匿名方法的地方都要避免捕获类的成员，小心内存泄漏</font>。

