# Java



## 一、设计模式

​	设计模式有三种类型23种

- 创建型模式：单例，抽象工厂，工厂，建造者，原型
- 结构型模式：适配器，桥接，装饰，组合，外观，享元，代理
- 行为型模式：模板方法，命令，迭代器，观察者，中介者，备忘录，解释器，状态，策略，职责链，访问者

### 1、创建型模式

#### 1）单例模式

##### （1）懒汉式

```java
/**
 * 懒汉式
 * 多线程下线程不安全
 * 是最容易想到的方法 只要构造器私有化就可以
 *
 * @author 为为
 * @create 9/3
 */
public class LazySingleton {

    private static LazySingleton instance;

    private LazySingleton() {}

    public static LazySingleton getInstance() {
        if(instance == null){
            instance = new LazySingleton();
        }
        return instance;
    }
}
```



##### （2）懒汉式线程安全版

```java
/**懒汉式加强
 * 多线程安全，但是因为加了sync 效率低下
 *
 * @author 为为
 * @create 9/3
 */
public class LazySingleton2 {

    private static LazySingleton2 instance ;

    private LazySingleton2(){}

    public static synchronized LazySingleton2 getInstance(){
        if(instance == null){
            instance = new LazySingleton2();
        }
        return instance;
    }
}
```

##### （3）饿汉式

```java
/**饿汉式
 * 线程安全
 * 只有在加载类的时候会new一个对象，利用了class的加载机制
 * 没有实现lazy loading，浪费内存
 * @author 为为
 * @create 9/3
 */
public class FullSingleton {

    private static FullSingleton instance = new FullSingleton();

    private FullSingleton(){}

    public static FullSingleton getInstance(){
        return instance;
    }
}
```

##### （4）双重检查锁

```java
/**双重检查锁
 * 线程安全，实现lazy loading
 * volatile 很关键，volatile表示标记的变量线程之间是共享的
 * 如果变量不加volatile，可能会得到半个实例
 * @author 为为
 * @create 9/3
 */
public class DCLSingleton {

    private static volatile DCLSingleton instance;

    private DCLSingleton() {
    }

    public static DCLSingleton getInstance() {
        if (instance == null) {
            synchronized (DCLSingleton.class) {
                if (instance == null) {
                    instance = new DCLSingleton();
                }
            }
        }
        return instance;
    }
}
```

##### （5）枚举

```java
/**
 * 枚举实现单例 最好的方式
 * 不怕反射 可以序列化（其他的方法 序列化和反序列化之后不是同一个对象）
 *
 * @author 为为
 * @create 9/3
 */
public enum  EnumSingleton {

    INSTANCE;
    public EnumSingleton getInstance(){
        return INSTANCE;
    }
}
```

#### 2）工厂模式

##### （1）简单工厂方法模式

> ​	提供一个公共的产品接口，实现接口的方法。写一个工厂类，接收不同参数，用if，else或者switch来判断返回那个产品类。

​	需要三个对象

- 产品的接口：定义公共的方法
- 多个产品的实现类
- 工厂类：有返回产品的方法，根据不同参数判断，返回不同的产品

​	这样做问题很多，如果需要加一个实现类，需要改动工厂的代码。

##### （2）工厂方法模式

> 提供一个公共的产品接口，产品类实现接口的方法。
>
> 提供一个公共的工厂接口，工厂类实现接口的方法。各个工厂类专门生产一种产品
>
> 需要三个对象

- 产品接口：
- 多个产品实现
- 工厂接口
- 多个工厂实现：不同的工厂返回不同的对象

​	这样生产产品不用接受参数，只需要使用不同的工厂类，来创建不同的产品，比简单工厂方法多了工厂接口类，多工厂的实现类。

​	这样如果加一个产品不需要改动代码，只需要添加一个产品的实现类和工厂的实现类

​	工厂方法模式有一个问题就是，类的创建依赖工厂类，也就是说，如果想要拓展程序，必须对工厂类进行修改，这违背了闭包原则

#### 3）抽象工厂模式

​	在工厂方法模式的基础上加上一个抽象工厂类，里面有所以工厂获得产品的方法

​	用户使用的时候只需要知道一个抽象工厂类就可以得到想要的产品

- 产品接口：
- 多个产品实现
- 工厂接口
- 多个工厂实现：不同的工厂返回不同的对象
- 抽象工厂类：可以获得不同的工厂，从而获得产品

#### 4）建造者模式

​	**定义**：将一个复杂对象的**构建**与它的**表示**分离，使得同样的构建过程可以创建不同的表示

​	与工厂模式的区别是：建造者模式更加关注与零件装配的顺序，一般用来创建更为复杂的对象

​	一个产品有很多组件，组件有不同的实现，想要相同部位的组件有不通过的功能，只要创建不同的组件实现类，设置产品的功能，返回产品

​	易于拓展，创建不同功能产品，只要写一个组件实现类就可以，屏蔽了复杂产品的实现过程，用户不知道实现过程。

​	使用范围有限，组件的的差异小，如电池的容量。产品差异大就不能用

- 产品类：产品对象，有组件的属性

- 产品组件接口类：有不同功能组件的定义，有一个返回产品的方法

- 产品组件实现类：实现组件接口的方法，有一个产品对象的属性，设置产品的组件，返回产品

- 指挥官类：传入不同的产品类，返回一个完整的产品

  使用时，调用指挥官的方法，传入想创建的产品类别，指挥官会调用产品类别组件的实现，设置到产品对象里，返回完整的产品。

  ```java
  
  /**手机产品
   * @author 为为
   * @create 9/6
   */
  public class Mobile {
  
      private String cpu;
      private String features;
  
      public String getCpu() {
          return cpu;
      }
  
      public void setCpu(String cpu) {
          this.cpu = cpu;
      }
  
      public String getFeatures() {
          return features;
      }
  
      public void setFeatures(String features) {
          this.features = features;
      }
  }
  
  
  /**组件接口类
   * @author 为为
   * @create 9/6
   */
  public interface Component {
  
      void buildCpu( );
      void buildFeatures( );
  
       Mobile createMobile();
  }
  
  
  /**组件的不同实现
   * @author 为为
   * @create 9/6
   */
  public class MeiZuMobile implements Component {
  
      private Mobile mobile;
      @Override
      public void buildCpu() {
          mobile.setCpu("1");
      }
  
      @Override
      public void buildFeatures() {
          mobile.setFeatures("2");
      }
  
  
      @Override
      public Mobile createMobile(){
          return mobile;
      }
  }
  
  /**指挥官
   * @author 为为
   * @create 9/6
   */
  public class Director {
  
      public Mobile createMobile(Component component){
          component.buildCpu();
          component.buildFeatures();
          return component.createMobile();
      }
  }
  ```

  

#### 5）原型模式

​	定义：用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

​	一个类有一个clone方法，返回与自身一样的结构和属性都一样的对象。实现一个接口 Cloneable接口，只是表示可以克隆，空接口。

​	Object p = p1;这种不是clone，只是p和p1指向同一个内存地址，是一个对象，clone是复制内存地址中的内容，从新开辟一块内存地址。

​	clone的时候，对象的构造方法不会执行，clone是从内存中已二进制的方式复制一份，所以不会执行构造方法。

​	clone是 浅度克隆，如果被克隆的对象有引用类型的属性，那么两个对象属性的引用是一样的。

​	深度克隆，就是把引用类型的属性，从新赋值给克隆对象，final修饰的属性不能深度克隆

​	可以提高效率，如果对象比较大的话，clone比new的效率高，clone是本地方法，从内存里读取二进制数据，可以逃避访问权限和构造器的约束。

### 2、结构型模式

#### 1）适配器模式

​	适配器模式的思想是：**把一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作**。

- 产品目标类：期待得到的目标

- 适配器：相当于转换头

- 产品源接口：期待的产品

  就是加强类，想要使用一个类的功能，但是这个类的功能不能完全满足自己的要求，就要有一个适配器类，实现完全功能接口的引用，持有被加强类的引用（或者继承），被加强类有的方法直接调用，没有的自己实现。

##### （1）类适配器模式

​	想要一个类拥有原本没有的方法，通过一个适配器类继承这个类，得到这个类的子类。

```java
public class Adaptee {
    public void method1(){
        System.out.println("method 1");
    }
}

public interface Target {
    void method1();
    void method2();
}

public class Adapter extends Adaptee implements Target { 
	@Override public void method2() { 
	System.out.println("method 2"); 
	} 
}

```

##### （2）对象适配器模式

​	**对象适配器模式是另外6种结构型设计模式的起源。** 

​	![img](C:\Users\Administrator\Desktop\MyNotes\src\java\images\SouthEast)

​	Adaptee类并没有method2()方法，而客户端则期待这个方法。与类适配器模式一样，为使客户端能够使用Adaptee类，我们把Adaptee与Target衔接起来。但这里我们不继承Adaptee，而是把Adaptee封装进Adapter里。这里Adaptee与Adapter是组合关系。 

```java
public class Adapter implements Target { 
	private Adaptee adaptee; 
	public Adapter(Adaptee adaptee) { 
		this.adaptee = adaptee; 
} 
	@Override public void method1() { 
		adaptee.method1(); 
	} 
	@Override public void method2() { 
		System.out.println("method 2"); 
	} 
} 

```

​	**类适配器与对象适配器的区别：类适配器使用的是继承的方式，直接继承了Adaptee，所以无法对Adaptee的子类进行适配。对象适配器使用的是组合的方式，·所以Adaptee及其子孙类都可以被适配。另外，对象适配器对于增加一些新行为非常方便，而且新增加的行为同时适用于所有的源。基于组合/聚合优于继承的原则，使用对象适配器是更好的选择。**

##### （3）接口适配器模式（缺省适配模式）

​	接口适配器模式（缺省适配模式）的思想是，为一个接口提供缺省实现，这样子类可以从这个缺省实现进行扩展，而不必从原有接口进行扩展。

​	如果需要使用一个接口里的一个方法，但是却要实现接口的所有方法，很麻烦，所以建议提高一个缺省的实现类，想要使用那个方法，只要重写那个方法就可以了。

#### 2）桥接模式

​	桥梁模式的用意是“将抽象化(Abstraction)与实现化(Implementation)脱耦，使得二者可以独立地变化

​	一个接口有多个不同实现类，定义一个桥类（产品类），持有接口的引用，构造的时候传入要创建的实现类，使用的时候直接调用实现类的方法。

​	JDBC就是桥接模式，DriverMananger可以传入不同的实现类，但是调用的时候都是一样的，不用改动代码。

- 功能接口
- 接口实现
- 桥：持有接口的引用，有传入实现类的方法，设置接口对象属性。

![http://dl2.iteye.com/upload/attachment/0083/1203/6f713d07-1409-3312-99c9-fa6b0909f0b2.jpg](C:\Users\Administrator\Desktop\MyNotes\src\java\images\6f713d07-1409-3312-99c9-fa6b0909f0b2.jpg)



#### 3）装饰模式

​	定义：是指允许对一个现有的对象加入其它额外的功能并且不改变其原来的结构

​	这实际上是基于对象适配器模式的一种变种。他与对象的适配器模式有异同点。 

​	相同点：都拥有一个目标对象

​	不同点：适配器模式需要实现另外一个接口，而装饰器模式必须实现该对象的接口。

- 功能接口：
- 功能实现类
- 装饰抽象类
- 装饰实现类

```java
/**功能接口
 * @author 为为
 * @create 9/6
 */
public interface MyRead {

    void read();
}

/**实现
 * @author 为为
 * @create 9/6
 */
public class MyFileRead implements MyRead {

    @Override
    public void read() {
        System.out.println("读取文件");
    }
}


/**实现
 * @author 为为
 * @create 9/6
 */
public class MyDbRead implements MyRead {

    @Override
    public void read() {
        System.out.println("读取内存");
    }
}


/**装饰抽象类
 * @author 为为
 * @create 9/6
 */
public  abstract class BuffRead implements MyRead {

    MyRead myRead;

    public BuffRead(MyRead myRead){
        this.myRead = myRead;
    }

    @Override
    public void read() {
        myRead.read();
    }
}


/**装饰实现类
 * @author 为为
 * @create 9/6
 */
public class BuffReader extends BuffRead {

    public BuffReader(MyRead myRead) {
        super(myRead);
    }
     @Override
    public void read() {
       System.out.println("读取前...");
        myRead.read();
        System.out.println("读取后...");
    }
    public void newRead(){
        System.out.println("新功能...");

    }
}
```



#### 4）组合模式

​	**组合模式：组合多个对象形成树形结构以表示有整体-部分关系层次结构，组合模式可以让客户端统一对待单个对象和组合对象**

​	文件管理系统就是一个组合模式，文件有两种类型，目录和文件。目录里可以有下级目录。文件就是结尾

（1）安全式组合模式

​	**安全性合成模式是指：**从客户端使用合成模式上看是否更安全，如果是安全的，那么就不会有发生误操作的可能，能访问的方法都是被支持的。

​	定义一个接口，接口里不定义操作子目录的方法，在目录的实现类里写。目录的实现类里有自身对象的引用，

（2）透明式组合模式

​	定义一个接口，接口里定义操作子目录的方法，但是因为文件没有操作的能力，所以实现是没有意义的。

#### 5）外观模式

​	**定义：外观模式是隐藏了系统的复杂性，能够为子系统中的一组接口提供一个统一的接口**

​	经常不经意间就使用了，一个类是单一职责，如果需要调用好几个类才能实现一个功能，那就可以另外写一个类，包装这个步骤，客户端调用的时候只需要一行代码。

#### 6）享元模式

​	**定义：运用共享技术有效地支持大量细粒度对象的复用。**

​	细粒度对象：是一些方法少，属性少的对象

​	就是对象的共享，一般和单例模式或者简单工厂模式配合使用

​	创建对象的工厂，工厂有一个map，获取对象的时候判断map里有没有，没有创建、放入map，下次就会从map中获得，对象的个数最多和实现类的个数一样。

#### 7）代理模式

##### （1）静态代理

​	目标实现了某一接口，代理类实现那个接口，持有被代理类的引用

```java
/**接口
 * @author 为为
 * @create 9/6
 */
public interface Sing {

    void sing();
}

/**实现
 * @author 为为
 * @create 9/6
 */
public class Singer implements Sing {

    @Override
    public void sing() {
        System.out.println("sing..........");
    }
}


/**代理
 * @author 为为
 * @create 9/6
 */
public class SingerProxy implements Sing {

    Singer singer;
    public SingerProxy(Singer singer){
        this.singer = singer;
    }

    @Override
    public void sing() {
        System.out.println("before.....");
        singer.sing();
        System.out.println("after.....");
    }
}
```

​	缺点是代理对象必须提前写出

##### （2）jdk（动态）代理

```java
/**接口
 * @author 为为
 * @create 9/6
 */
public interface Sing {

    void sing();
}

/**实现
 * @author 为为
 * @create 9/6
 */
public class Singer implements Sing {

    @Override
    public void sing() {
        System.out.println("sing..........");
    }
}

/**
 * @author 为为
 * @create 9/6
 */
public class TestProxy {

    public static void main(String[] args) {
        final Singer singer = new Singer();
        Sing instance = (Sing) Proxy.newProxyInstance(singer.getClass().getClassLoader(),
                singer.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("before....");
                        //这个传入的是上面new的对象，不是方法里的对象，需要设置为final
                        Object invoke = method.invoke(singer, args);
                        System.out.println("after....");
                        return invoke;
                    }
                });
        instance.sing();
    }
}
```



##### （3）CGilb代理

​	可以在被代理的类没有实现接口的时候使用。

```java
/**
 * @author 为为
 * @create 9/6
 */
public class Run {

    public void run(){
        System.out.println("run.....");
    }
}


/**
 * @author 为为
 * @create 9/6
 */
public class RunProxy implements MethodInterceptor {

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("before........");
        Object ret = methodProxy.invokeSuper(o, objects);
        System.out.println("after........");
        return ret;
    }
}

/**
 * @author 为为
 * @create 9/6
 */
public class Test {

    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Run.class);
        enhancer.setCallback(new RunProxy());
        Run run = (Run) enhancer.create();
        run.run();

    }
}
```



### 3、行为型模式

#### 1）模板方法模式

#### 2）命令模式

#### 3）迭代器模式

#### 4）观察者模式

​	定义：对象间一种**一对多**的依赖关系，使得每当一个对象改变状态，则所有**依赖它的对象**都会得到通知并**自动更新**。

- 被观察者
- 观察者接口
- 观察者实现类

​	思想比较常规：被观察者持有一组观察者的引用，只要实现接口就可以注册成为观察者，被观察者有改变想通知观察者时，遍历观察者引用，调用接口方法。

#### 5）中介者模式

​	定义：**中介者封装一系列对象相互作用，使得这些对象耦合松散，并且可以独立的改变它们之间的交互。**

​	常规思想：

#### 6）备忘录模式

#### 7）解释器模式

#### 8）状态模式

#### 9）策略模式

​	定义：策略模式（Strategy Pattern）属于对象的行为模式。其用意是针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以相互替换。策略模式使得算法可以在不影响到客户端的情况下发生变化。其主要目的是通过定义相似的算法，替换if else 语句写法，并且可以随时相互替换。

- 环境角色：持有接口的引用，可以调用接口的方法，

- 抽象策略角色：策略的接口

- 具体策略角色：实现类

  比较正常，定义一个相同的接口，有一些实现类，一个调用类，有一个传入接口的方法，可以传入不同的实现类。

  如果用常规的方法写一个简单计算器，用if、else来判断进行什么操作，用策咯模式，就定义所有操作的实现类，来创建不同的实现类来进行策略的选择。

#### 10）职责链模式

#### 11）访问者模式

#### 观察者模式

​	定义：在对象之间定义了一对多的依赖，这样一来，当一个对象改变状态，依赖它的对象会收到通知并自动更新

​	就是发布订阅模式，一个话题类里有所有关注者的引用，如果话题更新了调用通知关注者的方法。话题就是被观察者，用户是观察者，被观察者有什么变化，观察者可以接收到消息

​	需要4个角色

- 话题抽象类：需要有注册观察者的方法

- 用户抽象类：需要有接受通知的方法

- 话题实现类

- 用户实现类

  被观察者更新的时候，调用通知方法，调用用户接口的方法。

## 二、关键字

### transient

​	transient 作用是在bean实现了Serializable 接口时，序列化的时候，有些敏感属性（如：密码，身份证号）并不想序列化，可以在不想序列化的字段前边加上transient关键字,表示不序列化这个属性

### volatile

​	volatile 有两个作用

​	1、保持内存可见性

​	2、防止指令重排

​	为什么有这个作用呢?

​	因为变量的读和写都不是一步完成的，java通过下面的原子操作来完成主内存和工作内存的交互，完成变量的读和写

​	1、lock：作用于主内存，把变量标识为线程独占状态。

​	2、unlock：作用于主内存，解除独占状态。

​	3、read：作用主内存，把一个变量的值从主内存传输到线程的工作内存。

​	4、load：作用于工作内存，把read操作传过来的变量值放入工作内存的变量副本中。

​	5、use：作用工作内存，把工作内存当中的一个变量值传给执行引擎。

​	6、assign：作用工作内存，把一个从执行引擎接收到的值赋值给工作内存的变量。

​	7、store：作用于工作内存的变量，把工作内存的一个变量的值传送到主内存中。

​	8、write：作用于主内存的变量，把store操作传来的变量的值放入主内存的变量中。

​	![1567491873732](C:\Users\Administrator\Desktop\MyNotes\src\java\images\1567491873732.png)

​	volatile 的作用就是规定了 read、load、use 必须连续出现，assign、store、write必须连续出现

能够保证，每次读取前必会从主内存中得到最新值，每次写入后必会立即同步到主内存中，volatile修饰的变量线程间是可见的，每次得到的都是最新值

​	

```java
/**
 * 按照逻辑，线程在运行2秒之后，flag设为true，线程退出
 * 但是2秒之后打印出来了内容却没有退出JVM,说明flag还为false，但是主线程已经设置为true了
 * 这是因为vt里的flag是从工作内存中取出的，而不是从主内存中取出的，主线程更新的值，更新在主线程里面，所以
 * 设置了flag并不起作用
 * @author 为为
 * @create 9/3
 */
public class VolatileTest extends Thread {

    //volatile boolean flag = false;
    boolean flag = false;
    int i = 0;

    public void run() {
        while (!flag) {
            i++;
        }
    }

    public static void main(String[] args) throws Exception {
        VolatileTest vt = new VolatileTest();
        vt.start();
        Thread.sleep(2000);
        vt.flag = true;
        System.out.println("stope" + vt.i);
    }
}
```





​	防止指令重排在DCL中有很好的体现，DCL 的变量为什么需要volatile修饰，就是为了防止指令重排，从而得到半个对象，

​	instance = new Instance（）; 这个操作也不是原子性的，可以分为三步

​	1、memory = allocate();    //1：分配对象的内存空间

​	2、initInstance(memory);   //2：初始化对象

​	3、instance = memory;      //3：设置instance指向刚分配的内存地址

​	第2步依赖于第1步，但是第三步不依赖于第二步，所以JVM为了优化性能可能会进行重排序。可能的顺序为1-》3-》2; 如果到第三步结束，cpu切换到另一个线程，然后判断instance == null 为false ，返回instance ，但是这个instance只是半个对象，内存没有进行初始化。

### final

​	final可以修饰类、方法、成员变量，参数

​	当修饰类时，表示类不能被修改，就是不能继承

​	当修饰方法时，表示方法不能被重写（可以重载多个final修饰的方法），入股同时使用private修饰，会导致子类不能继承此方法，子类可以定义相同名字和参数的方法，不会与final冲突。**（注：类的private方法会隐式地被指定为final方法。）**

​	当修饰变量时，表示这是个常数，不能修改值，如果是对象，不能修改引用。本质上是一回事，因为引用的值是一个地址，final要求值，即地址的值不发生变化。当被final修饰的是基本数据类型以及String类型，如果在编译期就能确定值，JVM会把final当成编译期常量来处理。也就是说在用到该final变量的地方，相当于直接访问的这个值

```java
public class Test { 
    public static void main(String[] args)  { 
        String a = "hello2";   
        final String b = "hello"; 
        String d = "hello"; 
        String c = b + 2;   
        String e = d + 2; 
        System.out.println((a == c)); //true
        System.out.println((a == e)); //false
    } 
}
```

​	当修饰参数时，在方法里不能修改参数的值

### finally

​	finally是异常处理的一部分，与try、catch一起使用，通常会释放资源。不管有没有抛出异常，通常情况下一定会执行finally里的语句，如果在进入try之前就异常退出，虚拟机退出就不会执行finally里的语句。

​	finally里的return会覆盖try和catch里的return结果。

### **finalize**　　

​	finalize()是Object里的方法，gc回收资源的时候调用，一个对象的finalize只会调用一次，如果手动调用finalize，gc不会立即去回收，等到回收的时候可能又被引用了 ，这个对象就不能被回收了，会产生问题，所以不推荐手动调用finalize。

### native

​	native修饰表示这是一个本地方法，方法对应的实现不在当前文件，而是用其他语言实现的文件中，java语言本身不能对操作系统底层进行访问和操作，但是可以通过 JNI （Java Native Interface）接口来调用其它语言来访问底层

### strictfp

​	strictfp的意思是FP-strict，也就是说精确浮点的意思。在Java虚拟机进行浮点运算时，如果没有指定strictfp关键字时，Java的编译器以及运行环境在对浮点运算的表达式是采取一种近似于我行我素的行为来完成这些操作，以致于得到的结果往往无法令你满意。而一旦使用了strictfp来声明一个类、接口或者方法时，那么所声明的范围内Java的编译器以及运行环境会完全依照浮点规范IEEE-754来执行。因此如果你想让你的浮点运算更加精确，而且不会因为不同的硬件平台所执行的结果不一致的话，那就请用关键字strictfp。

## 三、内部类与外部类的互访

​	外部类访问内部类需要new内部类对象

​	内部类访问外部类可以直接访问，因为外部类持有内部类的引用

​	特例：内部类写在外部类的方法中(即局部变量的位置)

　　1、内部来外部类均可定义变量/常量

　　2、只能被final/abstract修饰

　　3、只能访问被final/abstract修饰的变量

　　4、可以直接访问外部类中的成员，因为还持有外部类的引用

### 1、内部类访问外部类

#### 1）非静态内部类的非静态方法

​	直接访问，

```java
　　public class Outter {
　　	int i = 5;
　　	static String string = "Hello";
　　	
　　	class Inner1{
　　		void Test1 (){
　　			System.out.println(i);
　　			System.out.println(string);
　　		}
　　	}
　　} 

```

#### 2）静态内部类的非静态方法

​	**因为静态方法访问非静态外部成员需先创建实例**，所以访问i时必须先new外部类。

```java
　　public class Outter {
　　	int i = 5;
　　	static String string = "Hello";
　　	
　　	static class Inner2{
　　		void Test1 (){
　　			System.out.println(new Outter().i);
　　			System.out.println(string);
　　		}
　　	}
　　}

```

#### 3）静态内部类的静态方法

​	必须先new外部类。

```java
　　public class Outter {
　　	int i = 5;
　　	static String string = "Hello";
　　	
　　	static class Inner2{
　　		static void Test2 (){
　　			System.out.println(new Outter().i);
　　			System.out.println(string);
　　		}
　　	}
　　} 

```

### 2、外部类访问内部类

#### 1）非静态方法访问非静态内部类的成员

​	new 内部类对象

```java
　　public class Outter {
　　	void Test1(){
　　		System.out.println(new Inner1().i);
　　	}
　　	class Inner1{
　　		int i = 5;
　　//		static String string = "Hello";  定义错误！
　　	}
　　}

```

#### 2）静态方法访问非静态内部类的成员

​	静态方法Test2访问非静态Inner1需先new外部类；

​	继续访问非静态成员i需先new 内部类

​	所以访问规则为：new Outter().new Inner1().i。

```java
　　public class Outter {
　　	static void Test2(){
　　		System.out.println(new Outter().new Inner1().i);
　　	}
　　	class Inner1{
　　		int i = 5;
　　//		static String string = "Hello";  定义错误！
　　	}
　　} 

```

#### 3）非静态方法访问静态内部类的成员

​	先“外部类.内部类”访问至内部类。

​	若访问静态成员，则需先new再访问；若访问非静态成员，则可直接访问。

```java
　　public class Outter {
　　	void Test1(){
　　		Outter.Inner2 inner2 = new Outter.Inner2();
　　		System.out.println(inner2.i);
　　		System.out.println(inner2.string);
　　		System.out.println(Outter.Inner2.string);
　　	}
　　	static class Inner2{
　　		int i = 5;
　　		static String string = "Hello"; 
　　	}
　　} 

```

#### 4）非静态方法访问非静态内部类的成员

​	先“外部类.内部类”访问至内部类，再new即可访问静态成员。

```java
　　public class Outter {
　　	void Test1(){
　　		Outter.Inner1 inner2 = new Outter.Inner1();
　　		System.out.println(inner2.i);
　　	}
　　	class Inner1{
　　		int i = 5;
　　//		static String string = "Hello"; 定义错误！
　　	}
　　}
```

#### 5）匿名内部类

​	匿名内部类访问外部成员变量时，变量前应加final关键字。

```java
		final int k = 6;
		((Button)findViewById(R.id.button2)).setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v) {
				// TODO Auto-generated method stub
				System.out.println(k);
			}
		});
```

## 四、Lambda

