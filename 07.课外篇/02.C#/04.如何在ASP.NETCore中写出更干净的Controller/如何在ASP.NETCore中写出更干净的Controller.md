转载：https://zhuanlan.zhihu.com/p/347004579

你可以遵循一些最佳实践来写出更干净的 Controller，一般我们称这种方法写出来的 Controller 为瘦Controller，瘦 Controller 的好处在于拥有更少的代码，更加单一的职责，也便于阅读和维护，而且随着时间的推移也容易做 Controller 的多版本。

这篇文章我们一起讨论那些让 Controler 变胖变臃肿的一些坏味道，并且一起探索让 Controller 变瘦的手段，虽然我的一些在 Controller 上的最佳实践可能不是专业的，但我每一步都提供相关源代码来进行优化。

接下来的章节中，我们会讨论什么是 胖Controller，什么是 坏味道，什么是 瘦Controller，它能带给我们什么福利？ 并且如何让 Controller 变瘦，变简单，利测试，易维护。

# 从 Controller 中移除数据层代码

当在写 Controller 的时候，你应该遵守 单一职责，也就意味着你的 Controller 只需做一件事情，换句话说，只有一个因素或者唯一一个因素能让你修改 Controller 中的代码，如果有点懵的话，考虑下面的代码片段，它将 数据访问代码 糅进了 Controller 。

```c#
public class AuthorController : Controller
{
    private AuthorContext dataContext = new AuthorContext();
    public ActionResult Index(int authorId)
    {
        var authors = dataContext.Authors
            .OrderByDescending(x=>x.JoiningDate)
            .Where(x=>x.AuthorId == authorId)
            .ToList();
        return View(authors);
    }
    //Other action methods
}
```

请注意上面的代码在 Action 中使用了 dataContext 从数据库读取数据，这就违反了单一职责原则，并直接导致了 Controller 的臃肿。

假如后续你需要修改 数据访问层 代码，可能基于更好的性能或者你能想到的原因，这时候只能被迫在 Controller 中修改，举个例子吧：假如你想把上面的 EF 改成 Dapper 去访问底层的 Database，更好的做法应该是单独拎出来一个 repository 类来操控 数据访问 相关的代码，下面是更新后的 AuthorController。

```c#
public class AuthorController : Controller
{
    private AuthorRepository authorRepository = new AuthorRepository();
    public ActionResult Index(int authorId)
    {
        var authors = authorRepository.GetAuthor(authorId);
        return View(authors);
    }
    //Other action methods
}
```

现在 AuthorController 看起来是不是精简多了，上面的代码是不是就是最佳实践呢？ 不完全是，为什么这么说呢？上面这种写法导致 Controller 变成了 数据访问组件，取出数据后必然少不了一些业务逻辑处理，这就让 Controller 违反了 单一职责，对吧，更通用的做法应该是将 数据访问逻辑 封装在一个 service 层，下面是优化之后的 AuthorController 类。

```c#
public class AuthorController : Controller
{
    private AuthorService authorService = new AuthorService();
    public ActionResult Index(int authorId)
    {
        var authors = authorService.GetAuthor(authorId);
        return View(authors);
    }
    //Other action methods
}
```

再看一下 AuthorService 类，可以看到它利用了 AuthorRepository 去做 CURD 操作。

```c#
public class AuthorService
{
    private AuthorRepository authorRepository = new AuthorRepository();
    public Author GetAuthor (int authorId)
    {
        return authorRepository.GetAuthor(authorId);
    }
    //Other methods
}
```

# 避免写大量代码做对象之间映射

在 DDD 开发中，经常会存在 DTO 和 Domain 对象，在数据 Input 和 Output 的过程中会存在这两个对象之间的 mapping，按照普通的写法大概就是这样的。

```c#
public IActionResult GetAuthor(int authorId)
{
    var author = authorService.GetAuthor(authorId);
    var authorDTO = new AuthorDTO();
    authorDTO.AuthorId = author.AuthorId;
    authorDTO.FirstName = author.FirstName;
    authorDTO.LastName = author.LastName;
    authorDTO.JoiningDate = author.JoiningDate;
    //Other code
   ......
}
```

可以看到，这种一一映射的写法让 Controller 即时膨胀，同时也让 Controller 增加了额外的功能，那如何把这种 模板式 代码规避掉呢？可以使用专业的 对象映射框架 AutoMapper 去解决，下面的代码展示了如何做 AutoMapper 的配置。

```c#
public class AutoMapping
{
    public static void Initialize()
    {
        Mapper.Initialize(cfg =>
        {
            cfg.CreateMap<Author, AuthorDTO>();
            //Other code            
        });
    }
}
```

接下来可以在 Global.asax 中调用 Initialize() 初始化，如下代码所示：

```c#
protected void Application_Start()
{
    AutoMapping.Initialize();         
}
```

最后，可以将 mapping 逻辑放在 service 层中，请注意下面的代码是如何使用 AutoMapper 实现两个不兼容对象之间的映射。

```c#
public class AuthorService
{
    private AuthorRepository authorRepository = new AuthorRepository();
    public AuthorDTO GetAuthor (int authorId)
    {
        var author = authorRepository.GetAuthor(authorId);
        return Mapper.Map<AuthorDTO>(author);
    }
    //Other methods
}
```

# 避免在 Controller 中写业务逻辑

尽量避免在 Controller 中写 业务逻辑 或者 验证逻辑， Controller 中应该仅仅是接收一个请求，然后被下一个 action 执行，别无其它，回到刚才的问题，这两种逻辑该怎么处理呢？

- 业务逻辑

这些逻辑可以封装 XXXService 类中，比如之前创建的 AuthorService。

- 验证逻辑

这些逻辑可以用 AOP 的操作手法，比如将其塞入到 Request Pipeline 中处理。

# 使用依赖注入而不是硬组合

推荐在 Controller 中使用依赖注入的方式来实现对象之间的管理，依赖注入是 控制反转 的一个子集，它通过外部注入对象之间的依赖从而解决内部对象之间的依赖，很拗口是吧！

一旦你用了依赖注入方式，就不需要关心对象是怎么实例化的，怎么初始化的，下面的代码展示了如何在 AuthorController 下的构造函数中实现 IAuthorService 对象的注入。

```c#
public class AuthorController : Controller
{
    private IAuthorService authorService = new AuthorService();
    public AuthorController(IAuthorService authorService)
    {
       this.authorService = authorService;
    }
   // Action methods
}
```

# 使用 action filer 消除 Controller 中的重复代码

可以利用 action filter 在 Request pipeline 这个管道的某些点上安插一些你的自定义代码，举个例子，可以使用 ActionFilter 在 Action 的执行前后安插一些自定义代码，而不是将这些业务逻辑放到 Controller 中，让 Controller 不必要的膨胀，下面的代码展示了如何去实现。

```c#
[ValidateModelState]
[HttpPost]
public ActionResult Create(AuthorRequest request)
{
    AuthorService authorService = new AuthorService();
    authorService.Save(request);
    return RedirectToAction("Home");
}
```

总的来说，如果一个 Controller 被赋予了几个职责，那么只要是其中任何一个职责的原因，你都必须对 Controller 进行修改，总的来说，一定要坚守 单一原则。
