因为类的实例是引用类型，要想用原有的类中的实例的数据的话，既要想创建原对象的一个副本的话,只能用clone方法。 

Clone方法分为深clone和浅clone 

# 浅Clone

在C#中提供了浅clone的方法，即为MemberwiseClone() 

实体类

```c#
public class Question : ObservableObject, ICloneable
{
    private string id = Guid.NewGuid().ToString();
    public string Id
    {
        get { return id; }
        set { id = value; RaisePropertyChanged(() => Id); }
    }

    public object Clone()
    {
        return this.MemberwiseClone();
    }
}
```

使用

```c#
var question=new Question();
var cloneQuestion = question.Clone() as Question;
```

MemberwiseClone()方法执行的只是浅层拷贝。

# 深Clone

深层拷贝要递归的拷贝其字段所引用的所有对象。

**深克隆：要在它的每一个包含的类中实现浅Clone**

```c#
public class Question : ObservableObject, ICloneable
{
    private string id = Guid.NewGuid().ToString();
    public string Id
    {
        get { return id; }
        set { id = value; RaisePropertyChanged(() => Id); }
    }

    private List<QuestionSelection> questionSelections = new List<QuestionSelection>()
    {
        new QuestionSelection{ Content="A",IsChecked=false,},
        new QuestionSelection{ Content="B",IsChecked=false,},
        new QuestionSelection{ Content="C",IsChecked=false,},
        new QuestionSelection{ Content="D",IsChecked=false,},
    };
    public List<QuestionSelection> QuestionSelections
    {
        get { return questionSelections; }
        set { questionSelections = value; RaisePropertyChanged(() => QuestionSelections); }
    }

    public object Clone()
    {
        var question = this.MemberwiseClone() as Question;

        var questionSelections=new List<QuestionSelection>();
        
        foreach(var qs in question.QuestionSelections){
            questionSelections.Add(qs.Clone as QuestionSelection);
        }

        question.QuestionSelections = questionSelections;

        return question;
    }
}

public class QuestionSelection : ObservableObject, ICloneable
{
    private string content;
    public string Content
    {
        get { return content; }
        set { content = value; RaisePropertyChanged(() => Content); }
    }

    private bool isChecked;
    public bool IsChecked
    {
        get { return isChecked; }
        set { isChecked = value; RaisePropertyChanged(() => IsChecked); }
    }

    private string id;
    public string Id
    {
        get { return id; }
        set { id = value; RaisePropertyChanged(() => Id); }
    }

    public object Clone()
    {
        return this.MemberwiseClone();
    }
}
```