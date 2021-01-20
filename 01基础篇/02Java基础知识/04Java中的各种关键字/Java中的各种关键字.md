# transient

ArrayList类和Vector类都是使用数组实现的，但是在定义数组elementData这个属性时稍有不同，那就是ArrayList使用transient关键字

```java
private transient Object[] elementData;  

protected Object[] elementData;  
```

Java语言的关键字，变量修饰符，**如果用transient声明一个实例变量，当对象存储时，它的值不需要维持**。这里的对象存储是指，Java的serialization提供的一种持久化对象实例的机制。当一个对象被序列化的时候，transient型变量的值不包括在序列化的表示中，然而非transient型的变量是被包括进去的。使用情况是：**当持久化对象时，可能有一个特殊的对象数据成员，我们不想用serialization机制来保存它。为了在一个特定对象的一个域上关闭serialization，可以在这个域前加上关键字transient。**

简单点说，**就是被transient修饰的成员变量，在序列化的时候其值会被忽略，在被反序列化后， transient 变量的值被设为初始值， 如 int 型的是 0，对象型的是 null。**

# instanceof

instanceof 是 Java 的一个二元操作符，类似于 ==，>，< 等操作符。

instanceof 是 Java 的保留关键字。它的作用是**测试它左边的对象是否是它右边的类的实例，返回 boolean 的数据类型**。

以下实例创建了 displayObjectClass() 方法来演示 Java instanceof 关键字用法：

```java
public static void displayObjectClass(Object o) {
  if (o instanceof Vector)
     System.out.println("对象是 java.util.Vector 类的实例");
  else if (o instanceof ArrayList)
     System.out.println("对象是 java.util.ArrayList 类的实例");
  else
    System.out.println("对象是 " + o.getClass() + " 类的实例");
}
```

# volatile

Java语言为了解决并发编程中存在的原子性、可见性和有序性问题，提供了一系列和并发处理相关的关键字，比如synchronized、volatile、final、concurren包等

## volatile的用法

volatile通常被比喻成"轻量级的synchronized"，也是Java并发编程中比较重要的一个关键字。和synchronized不同，volatile是一个变量修饰符，只能用来修饰变量。无法修饰方法及代码块等。

volatile的用法比较简单，只需要在声明一个可能被多线程同时访问的变量时，使用volatile修饰就可以了

```java
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}
```

## volatile的原理

为了提高处理器的执行速度，在处理器和内存之间增加了多级缓存来提升。但是由于引入了多级缓存，就存在缓存数据不一致问题。

对于volatile变量，当对volatile变量进行写操作的时候，JVM会向处理器发送一条lock前缀的指令，将这个缓存中的变量回写到系统主存中。

但是就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题，所以在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议

**缓存一致性协议：每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器要对这个数据进行修改操作的时候，会强制重新从系统内存里把数据读到处理器缓存里。**

所以，**如果一个变量被volatile所修饰的话，在每次数据变化之后，其值都会被强制刷入主存**。而其他处理器的缓存由于遵守了缓存一致性协议，也会把这个变量的值从主存加载到自己的缓存中。这就保证了一个volatile在并发编程中，其值在多个缓存中是可见的。

## volatile与可见性

可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

Java内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。所以，就可能出现线程1改了某个变量的值，但是线程2不可见的情况。

前面的关于volatile的原理中介绍过了，Java中的volatile关键字提供了一个功能，那就是被其修饰的变量在被修改后可以立即同步到主内存，被其修饰的变量在每次使用之前都从主内存刷新。因此，**可以使用volatile来保证多线程操作时变量的可见性**。

## volatile与有序性

有序性即程序执行的顺序按照代码的先后顺序执行。

除了引入了时间片以外，由于处理器优化和**指令重排**等，CPU还可能对输入代码进行乱序执行，比如load->add->save 有可能被优化成load->save->add 。这就是可能存在有序性问题。

而volatile除了可以保证数据的可见性之外，还有一个强大的功能，那就是他可以**禁止指令重排优化**等

普通的变量仅仅会保证在该方法的执行过程中所依赖的赋值结果的地方都能获得正确的结果，而不能保证变量的赋值操作的顺序与程序代码中的执行顺序一致。

volatile可以禁止指令重排，这就保证了代码的程序会严格按照代码的先后顺序执行。这就保证了有序性。被volatile修饰的变量的操作，会严格按照代码顺序执行，load->add->save 的执行顺序就是：load、add、save。

就是**防止指令重重排**

## volatile与原子性

原子性是指**一个操作是不可中断的，要全部执行完成，要不就都不执行**。

线程是CPU调度的基本单位。CPU有时间片的概念，会根据不同的调度算法进行线程调度。当一个线程获得时间片之后开始执行，在时间片耗尽之后，就会失去CPU使用权。所以在多线程场景下，由于时间片在线程间轮换，就会发生原子性问题。

我们介绍synchronized的时候，提到过，**为了保证原子性，需要通过字节码指令monitorenter和monitorexit**，但是volatile和这两个指令之间是没有任何关系的。

所以，**volatile是不能保证原子性的**


在以下两个场景中可以使用volatile来代替synchronized：

- 1、运算结果并不依赖变量的当前值，或者能够确保只有单一的线程会修改变量的值。
- 2、变量不需要与其他状态变量共同参与不变约束。

其他情况下都需要使用其他方式来保证原子性，如synchronized或者concurrent包

volatile和原子性的例子

```java
public class Test {
    public volatile int inc = 0;

    public void increase() {
        inc++;
    }

    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }

        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

## 总结

**synchronized可以保证原子性、有序性和可见性**。而**volatile却只能保证有序性和可见性**。

# synchronized

## 概述

synchronized关键字在需要原子性、可见性和有序性这三种特性的时候都可以作为其中一种解决方案，看起来是“万能”的。的确，大部分并发控制操作都能使用synchronized来完成。

## 用法

synchronized是Java提供的一个并发控制的关键字。主要有两种用法，分别是同步方法和同步代码块。也就是说，synchronized既可以修饰方法也可以修饰代码块。

```java
/**
 * @author Hollis 18/08/04.
 */
public class SynchronizedDemo {
     //同步方法
    public synchronized void doSth(){
        System.out.println("Hello World");
    }

    //同步代码块
    public void doSth1(){
        synchronized (SynchronizedDemo.class){
            System.out.println("Hello World");
        }
    }
}
```
被synchronized修饰的代码块及方法，在同一时间，只能被单个线程访问。

## 实现原理

对上面的代码进行反编译，可以得到如下代码

```java
public synchronized void doSth();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Hello World
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return

  public void doSth1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #5                  // class com/hollis/SynchronizedTest
         2: dup
         3: astore_1
         4: monitorenter
         5: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         8: ldc           #3                  // String Hello World
        10: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        13: aload_1
        14: monitorexit
        15: goto          23
        18: astore_2
        19: aload_1
        20: monitorexit
        21: aload_2
        22: athrow
        23: return
```

对于同步方法，**JVM采用ACC_SYNCHRONIZED标记符来实现同步**。 **对于同步代码块。JVM采用monitorenter、monitorexit两个指令来实现同步**。

**方法级的同步是隐式的**。**同步方法的常量池中会有一个ACC_SYNCHRONIZED标志**。**当某个线程要访问某个方法的时候，会检查是否有ACC_SYNCHRONIZED，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁**。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。**值得注意的是，如果在方法执行过程中，发生了异常，并且方法内部并没有处理该异常，那么在异常被抛到方法外面之前监视器锁会被自动释放**。

**同步代码块使用monitorenter和monitorexit两个指令实现**。**可以把执行monitorenter指令理解为加锁，执行monitorexit理解为释放锁**。 **每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行monitorenter）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行monitorexit指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。**

无论是ACC_SYNCHRONIZED还是monitorenter、monitorexit都是**基于Monitor实现的**，**在Java虚拟机(HotSpot)中，Monitor是基于C++实现的，由ObjectMonitor实现**

ObjectMonitor类中提供了几个方法，如enter、exit、wait、notify、notifyAll等。**sychronized加锁的时候，会调用objectMonitor的enter方法，解锁的时候会调用exit方法**

## 与原子性

原子性是指一个操作是不可中断的，要全部执行完成，要不就都不执行

线程是CPU调度的基本单位。CPU有时间片的概念，会根据不同的调度算法进行线程调度。当一个线程获得时间片之后开始执行，在时间片耗尽之后，就会失去CPU使用权。所以在多线程场景下，由于时间片在线程间轮换，就会发生原子性问题。

在Java中，为了保证原子性，提供了两个高级的字节码指令monitorenter和monitorexit。前面中，介绍过，这两个字节码指令，在Java中对应的关键字就是synchronized。

通过monitorenter和monitorexit指令，可以保证被synchronized修饰的代码在同一时间只能被一个线程访问，在锁未释放之前，无法被其他线程访问到。因此，**在Java中可以使用synchronized来保证方法和代码块内的操作是原子性的**

线程1在执行monitorenter指令的时候，会对Monitor进行加锁，加锁后其他线程无法获得锁，除非线程1主动解锁。即使在执行过程中，由于某种原因，比如CPU时间片用完，**线程1放弃了CPU，但是，他并没有进行解锁**。**而由于synchronized的锁是可重入的，下一个时间片还是只能被他自己获取到，还是会继续执行代码**。直到所有代码执行完。这就保证了原子性。

## 与可见性

可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

Java内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的变量的主内存副本拷贝，线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。所以，就可能出现线程1改了某个变量的值，但是线程2不可见的情况。

前面我们介绍过，被synchronized修饰的代码，在开始执行时会加锁，执行完成后会进行解锁。而为了保证可见性，有一条规则是这样的：**对一个变量解锁之前，必须先把此变量同步回主存中。这样解锁后，后续线程就可以访问到被修改后的值。**

synchronized关键字锁住的对象，其值是**具有可见性的**

## 与有序性

有序性即程序执行的顺序按照代码的先后顺序执行。

除了引入了时间片以外，由于处理器优化和指令重排等，CPU还可能对输入代码进行乱序执行，比如load->add->save 有可能被优化成load->save->add 。这就是可能存在有序性问题。

**这里需要注意的是，synchronized是无法禁止指令重排和处理器优化的。也就是说，synchronized无法避免上述提到的问题。**

为什么还说synchronized也提供了有序性保证呢？

这就要再把有序性的概念扩展一下了。Java程序中天然的有序性可以总结为一句话：如果在本线程内观察，所有操作都是天然有序的。如果在一个线程中观察另一个线程，所有操作都是无序的。

这里我简单扩展一下，这其实和as-if-serial语义有关

as-if-serial语义的意思指：**不管怎么重排序（编译器和处理器为了提高并行度），单线程程序的执行结果都不能被改变。编译器和处理器无论如何优化，都必须遵守as-if-serial语义**

**as-if-serial语义保证了单线程中，指令重排是有一定的限制的，而只要编译器和处理器都遵守了这个语义，那么就可以认为单线程程序是按照顺序执行的。当然，实际上还是有重排的，只不过我们无须关心这种重排的干扰**

**由于synchronized修饰的代码，同一时间只能被同一线程访问。那么也就是单线程执行的。所以，可以保证其有序性。**

就是在广义上，单线程最后得到的结果是不变的，所以是有序的，广义上的有序

## 与锁优化

synchronized其实是借助Monitor实现的，在加锁时会调用objectMonitor的enter方法，解锁的时候会调用exit方法。事实上，只有在JDK1.6之前，synchronized的实现才会直接调用ObjectMonitor的enter和exit，这种锁被称之为重量级锁。

在JDK1.6中出现对锁进行了很多的优化，进而出现轻量级锁，偏向锁，锁消除，适应性自旋锁，锁粗化(自旋锁在1.4就有，只不过默认的是关闭的，jdk1.6是默认开启的)，这些操作都是为了在线程之间更高效的共享数据 ，解决竞争问题。

# final

final是Java中的一个关键字，它所表示的是“这部分是无法修改的”

使用 final 可以定义 ：变量、方法、类

- final变量，一旦final变量被定义之后，是无法进行修改的
- final方法，如果任何方法声明为final，则不能覆盖它
- final类，如果把任何一个类声明为final，则不能继承它

# static

static表示“静态”的意思，用来修饰成员变量和成员方法，也可以形成静态static代码块

## 静态变量

我们用static表示变量的级别，一个类中的静态变量，不属于类的对象或者实例。因为静态变量与所有的对象实例共享，因此他们不具线程安全性

通常，静态变量常用final关键来修饰，表示通用资源或可以被所有的对象所使用。如果静态变量未被私有化，可以用“类名.变量名”的方式来使用

## 静态方法

与静态变量一样，静态方法是属于类而不是实例

一个静态方法只能使用静态变量和调用静态方法。通常静态方法通常用于想给其他的类使用而不需要创建实例。例如：Collections class(类集合)

Java的包装类和实用类包含许多静态方法。main()方法就是Java程序入口点，是静态方法

从Java8以上版本开始也可以有接口类型的静态方法了

## 静态代码块

Java的静态块是一组指令在类装载的时候在内存中由Java ClassLoader执行

静态块常用于初始化类的静态变量。大多时候还用于在类装载时候创建静态资源

Java不允许在静态块中使用非静态变量。一个类中可以有多个静态块，尽管这似乎没有什么用。静态块只在类装载入内存时，执行一次

示例

```java
static{
    //can be used to initialize resources when class is loaded
    System.out.println("StaticExample static block");
    //can access only static variables and methods
    str="Test";
    setCount(2);
}
```

## 静态类

Java可以嵌套使用静态类，但是静态类不能用于嵌套的顶层

静态嵌套类的使用与其他顶层类一样，嵌套只是为了便于项目打包

# const

const是Java预留关键字，用于后期扩展用，用法跟final相似，不常用