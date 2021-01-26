转载：https://www.cnblogs.com/fred-bao/p/5700776.html

# 前言

通常在一个应用程序中，我们开发人员会在两个不同的类型对象之间传输数据，通常我们会用DTOs(数据传输对象)，View Models(视图模型)，或者直接是一些从一个service或者Web API的一些请求或应答对象。一个常见的需要使用数据传输对象的情况是，我们想把属于一个对象的某些属性值赋值给另一个对象的某些属性值，但是问题是，这个两个对象可能并不是完全匹配的，比如，两者之间的属性类型，名称等等，是不一样的，或者我们只是想把一个对象的一部分属性值赋值给另一个对象。


# 手动映射

首先，让我们来看下之前的处理方式，我们通过以下这个例子来直观感受这种方式，我们创建了以下三个类：

```c#
public class Author
{
    public string Name { get; set; }
}

public class Book
{
    public string Title { get; set; }
    public Author Author { get; set; }
}

public class BookViewModel
{
    public string Author { get; set; }
}
```

为了创建Book对象实例的一个View Model对象实例-BookViewModel对象实例，我们需要写如下代码：

```c#
BookViewModel model = new BookViewModel
{
    Title = book.Title,
    Author = book.Author.Name
}
```

上面的例子相当的直观了，但是问题也随之而来了，我们可以看到在上面的代码中，如果一旦在Book对象里添加了一个额外的字段，而后想在前台页面输出这个字段，那么就需要去在项目里找到每一处有这样转换字段的地方，这是非常繁琐的。

另外，BookViewModel.Author是一个string类型的字段,但是Book.Author属性却是Author对象类型的，我们用的解决方法是通过Book.Auther对象来取得Author的Name属性值，然后再赋值给BookViewModel的Author属性，这样看起行的通，但是想一想，如果打算在以后的开发中把Name拆分成两个-FisrtName和LastName,那么，呵呵，我们得去把原来的ViewModel对象也拆分成对应的两个字段，然后在项目中找到所有的转换，然后替换。

那么有什么办法或者工具来帮助我们能够避免这样的情况发生呢？AutoMapper正是符合要求的一款插件。

# 使用AutoMapper

到现在，确切的说，AutoMapper的安装使用非常非常的便捷，就如同傻瓜照相机那样。你只需要从Nuget上下载AutoMapper的包到你的应用程序里，然后添加对AutoMapper命名空间的引用，然后你就可以在你的项目里随意使用它了。以下就是一个非常简单的是例子:

```c#
AutoMapper.Mapper.CreateMap<Book, BookViewModel>();
var model = AutoMapper.Mapper.Map<BookViewModel>(book);
```

使用AutoMappeer的好处是显而易见的，首先，不再需要我们去对DTO实例的属性一一赋值，然后无论你在Book对象或者BookViewModel对象里加了一个或者更多的字段，那都不会影响这个段映射的代码，我不再需要去找到每一处转换的地方去更改代码，你的程序会像之前正常运转。 

不过，还是有个问题并没有得到很好的解决，这也是在AutoMapper文档上缺失的，为把Book.Athor.Name字段赋值给BookViewModel.Author字段，需要在每一处需要执行映射的代码地方，同时创建一个如下的显示转换申明代码，所以如果有很多处转换的话，那么我们就会写很多重复的这几行代码:

```c#
AutoMapper.Mapper.CreateMap<Book, BookViewModel>().ForMember(dest => dest.Author,opts => opts.MapFrom(src => src.Author.Name));
```

所以我们该如何正确的创建映射呢？方式有很多，我这边说下在ASP.NET MVC的程序里如何处理。

在微软的ASP.NET MVC程序中，它提供了一个Global.asax文件，这个文件里可以放置一些全剧配置，上面对于把Book.Athor.Name字段赋值给BookViewModel.Author字段这个映射配置放置在这个文件里面，那么这段代码只会跑一次但是所有转换的地方都能正确的转换Book.Athor.Name为BookViewModel.Author。

当然，Global.asax文件中不建议放很复杂的代码，因为这是ASP.NET程序的入口，一档这个文件里出错，那么整个程序就会over。配置代码可以以这样的形式写，创建一个AutoMapper的配置类:

```c#
public static class AutoMapperConfig
{
    public static void RegisterMappings()
    {
        AutoMapper.Mapper.CreateMap<Book, BookViewModel>().ForMember(dest => dest.Author,opts => opts.MapFrom(src => src.Author.Name));
    }
}
```

然后再Global文件注册这个类:

```c#
protected override void Application_Start(object sender, EventArgs e)
{
    AutoMapperConfig.RegisterMappings();
}
```

# 创建映射

所有的映射是有CreateMap方法来完成的：

```c#
AutoMapper.Mapper.CreateMap<SourceClass, >();
```

需要注意的是：这种方式是单向的匹配，即在在创建了上面的映射了之后我们可以在程序里从一个SourceClass实例得到一个DestinationClass类型的对象实例：

```c#
var destinationClass= AutoMapper.Mapper.Map<DestinationClass>(sourceClass);
```

但是如果尝试从DestinationClass映射到一个SourceClass，我们到的是一个错误信息：

```c#
var book = AutoMapper.Mapper.Map<Book>(bookViewModel);
```

幸运的是，AutoMapper已经考虑到这个问题了，它提供了ReverseMap方法：

```c#
AutoMapper.Mapper.CreateMap<Book, BookViewModel>().ReverseMap();
```

使用了这个方式后你就可以从Book创建BookViewModel，同时也可以从BookViewModel创建Book对象实例。

# Conventions

AutoMapper之所以能和任何一种集合类型产生交集，是由于它可以配置各种Conventions来完成一个类型到另一个类型的映射。最基本的一点就是两个映射类型之间的字段名称需要相同。例如一下的一个例子:

```c#
public class Book
{
    public string Title { get; set; }
}

public class NiceBookViewModel
{
    public string Title { get; set; }
}

public class BadBookViewModel
{
    public string BookTitle { get; set; }
}
```

如果从Book映射到NiceBookViewModel，那么NiceBookBiewModel的Title属性会被正确设置，但是如果将Book映射为BadBookViewModel，那么BookTitle的属性值将会为NULL值。

所以这种情况下，AutoMapper看起来失效了，不过，幸运的是，AutoMapper已经预先考虑到这种情况了，AutoMapper可以通过投影的方式来正确的映射BadBookViewModel和Book，只需要一行代码：

```c#
AutoMapper.Mapper.CreateMap<Book, BadBookViewModel>().ForMember(dest => dest.BookTitle,opts => opts.MapFrom(src => src.Title));
```

一种比较复杂的情况的是，当一个类型中引用了另一个类型的作为其一个属性，例如：

```c#
public class Author
{
    public string Name { get; set; }
}

public class Book
{
    public string Title { get; set; }
    public Author Author { get; set; }
}

public class BookViewModel
{
    public string Title { get; set; }
    public string Author { get; set; }
}
```

虽然Book和BookViewModel都有这一个Author的属性子都，但是它们的类型是不同，所有如果使用AutoMapper来映射Book的Author到BookViewModel的Author，我们得到的还是一个NULL值。

对于这种以另一个类型为属性的映射，AutoMapper内置默认的有个Conventions是会这个的属性名加上这个属性的类型里的属性名称映射到目标类型具有相同名称的字段，即如果在BookViewModel里有一个叫AuthorName的，那么我们可以得到正确的Name值。但是如果我们既不想改名称，又想能正确的映射，怎么办呢？Convention就是为此而诞生的：

```c#
AutoMapper.Mapper.CreateMap<Book, BookViewModel>().ForMember(dest => dest.Author,opts => opts.MapFrom(src => src.Author.Name));
```

对于AutoMapper，它提供的Conventions功能远不止这些，对于更加复杂的情形，它也能够应对，例如当Author类型的字段有两个属性组成：

```c#
public class Author
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

但是我们仍然只想映射到BookViewModel的一个字段，为此，我们可以这么做：

```c#
AutoMapper.Mapper.CreateMap<Book, BookViewModel>().ForMember(dest => dest.Author,opts => opts.MapFrom(src => string.Format("{0} {1}",src.Author.FirstName,src.Author.LastName)));
```

还可以更加复杂，例如：

```c#
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string ZipCode { get; set; }
}

public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public Address Address { get; set; }
}

public class PersonDTO
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Street { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string ZipCode { get; set; }
}
```

如果从Person映射为PersonDTO，我们只要想上面一样的做饭就可以了。但是如果这个时候我们要做的是把PersonDTO映射为Book实体呢？代码其实是差不多的：

```c#
AutoMapper.Mapper.CreateMap<PersonDTO, Person>().ForMember(dest => dest.Address,opts => opts.MapFrom(src => new Address{Street = src.Street,City = src.City,State = src.State,ZipCode = src.ZipCode}));
```

所以，我们在Convertion中构建了一个新的Address的实例，然后赋值给Book的Address的属性。 
有时候，我们可能创建了不止一个DTO来接受映射的结果，例如，对于Address，我们同样创建了一个AddressDTO：

```c#
public class AddressDTO
{
    public string Street { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string ZipCode { get; set; }
}

public class PersonDTO
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public AddressDTO Address { get; set; }
}
```

这个时候如果我们直接尝试把Person映射为PersonDTO，会报错，映射AutoMapper并不知道Address和AddressDTO之间的映射关系，我们需要手动创建：

```c#
AutoMapper.Mapper.CreateMap<PersonDTO, Person>();
AutoMapper.Mapper.CreateMap<AddressDTO, Address>();
```

# 映射到一个已存在的实例对象

之前我们都是把映射得到的结果赋值给一个变量，AutoMapper提供了另外一种方式，它使得我们可以直接映射两个已存在的实例。 

之前的做法:

```c#
AutoMapper.Mapper.CreateMap<SourceClass, DestinationClass>();
var destinationObject = AutoMapper.Mapper.Map<DestinatationClass>(sourceObject);
```

直接映射的做法：

```c#
AutoMapper.Mapper.Map(sourceObject, destinationObject);
```

AutoMapper也支持映射集合对象：

```c#
var destinationList = AutoMapper.Mapper.Map<List<DestinationClass>>(sourceList);
```

对于ICollectionIEnumerable的也是同样适用。

但是在用AutoMapper来实现内部的集合映射的时候，是非常非常不愉快的，因为AutoMapper会把这个集合作为一个属性来映射赋值，而不是把内置的集合映里的一行行内容进行射，例如对于如下的一个例子：

```c#
public class Pet
{
    public string Name { get; set; }
    public string Breed { get; set; }
}

public class Person
{
    public List<Pet> Pets { get; set; }
}


public class PetDTO
{
    public string Name { get; set; }
    public string Breed { get; set; }
}

public class PersonDTO
{
    public List<PetDTO> Pets { get; set; }
}
```

我们在页面上创建一个更新Pet类型的Name属性的功能，然后提交更新，收到的数据差不多是这样：

```c#
{
    Pets: [
        { Name : "Sparky", Breed : null },
        { Name : "Felix", Breed : null },
        { Name : "Cujo", Breed : null }
    ]
}
```

这个时候如果我们去将Person映射为PersonDTO：

```c#
AutoMapper.Mapper.Map(person, personDTO);
```

我们得到将是一个全新的Pet的集合，即Name是更新后的数据，但是所有的Breed的值都将为NULL，这个不是所期望的结果。

很不幸的是，AutoMapper并没有提供很好的解决方案。

目前能做的一种方案就是用AutoMapper的Ignore方法忽略Pet的属性的映射，然后我们自己去完成映射：

```c#
AutoMapper.Mapper.CreateMap<PersonDTO, Person>().ForMember(dest => dest.Pets,opts => opts.Ignore());
```

```c#
AutoMapper.Mapper.Map(person, personDTO);
for (int i = 0; i < person.Pets.Count(); i++)
{
    AutoMapper.Mapper.Map(person.Pets[i], personDTO.Pets[i]);
}
```