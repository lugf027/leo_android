# Java_Proxy 

## 1、简介



## 2、代理模式(静态代理)

### 2.1  简介

由于某些原因需要给某对象提供一个代理以控制对该对象的访问。这时，访问对象不适合或者不能直接引用目标对象，代理对象作为访问对象和目标对象之间的中介。

* 主要优点：

  - 代理模式在客户端与目标对象之间起到一个中介作用和保护目标对象的作用；

  - 代理对象可以扩展目标对象的功能；

  - 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度，增加了程序的可扩展性

* 主要缺点：

  - 代理模式会造成系统设计中类的数量增加

  - 在客户端和目标对象之间增加一个代理对象，会造成请求处理速度变慢；

  - 增加了系统的复杂度；

> 可以使用**动态代理**方式解决上述缺点

### 2.2 代理模式结构

代理模式的结构比较简单，主要是通过定义一个继承抽象主题的代理来包含真实主题，从而实现对真实主题的访问。

1. 抽象主题（Subject）类：通过接口或抽象类声明真实主题和代理对象实现的业务方法。
2. 真实主题（Real Subject）类：实现了抽象主题中的具体业务，是代理对象所代表的真实对象，是最终要引用的对象。
3. 代理（Proxy）类：提供了与真实主题相同的接口，其内部含有对真实主题的引用，它可以访问、控制或扩展真实主题的功能。

![](./imgs/java_proxy_design.gif)

在代码中，一般代理会被理解为代码增强，实际上就是在原代码逻辑前后增加一些代码逻辑，而使调用者无感知。

根据代理的创建时期，代理模式分为静态代理和动态代理。

- **静态**：由程序员创建代理类或特定工具自动生成源代码再对其编译，在程序运行前代理类的 .class 文件就已经存在了。
- **动态**：在程序运行时，运用反射机制动态创建而成

### 2.2 代理模式示例

```Java
public class ProxyTest {
    public static void main(String[] args) {
        SubjectImplProxy subjectImplProxy = new SubjectImplProxy();
        subjectImplProxy.request();
    }
}

// 抽象主题
interface ISubject {
    void request();
}

// 真实主题
class SubjectImpl implements ISubject {
    public void request() {
        System.out.println("访问真实主题方法...");
    }
}

// 静态代理
class SubjectImplProxy implements ISubject {
    private SubjectImpl subjectImpl;

    public void request() {
        if (subjectImpl == null) {
            subjectImpl = new SubjectImpl();
        }
        preRequest();
        subjectImpl.request();
        postRequest();
    }

    public void preRequest() {
        System.out.println("访问真实主题之前的预处理。");
    }

    public void postRequest() {
        System.out.println("访问真实主题之后的后续处理。");
    }
}
```

程序运行的结果如下：

```shell
访问真实主题之前的预处理。
访问真实主题方法...
访问真实主题之后的后续处理。
```

### 2.4 代理模式应用场景

使用代理模式主要有两个目的：一是保护目标对象，二是增强目标对象。应用场景：

- 远程代理，这种方式通常是为了隐藏目标对象存在于不同地址空间的事实，方便客户端访问。例如，用户申请某些网盘空间时，会在用户的文件系统中建立一个虚拟的硬盘，用户访问虚拟硬盘时实际访问的是网盘空间。
- 虚拟代理，这种方式通常用于要创建的目标对象开销很大时。例如，下载一幅很大的图像需要很长时间，因某种计算比较复杂而短时间无法完成，这时可以先用小比例的虚拟代理替换真实的对象，消除用户对服务器慢的感觉。
- 安全代理，这种方式通常用于控制不同种类客户对真实对象的访问权限。
- 智能指引，主要用于调用目标对象时，代理附加一些额外的处理功能。例如，增加计算真实对象的引用次数的功能，这样当该对象没有被引用时，就可以自动释放它。
- 延迟加载，指为了提高系统的性能，延迟对目标的加载。例如，[Hibernate](http://c.biancheng.net/hibernate/) 中就存在属性的延迟加载和关联表的延时加载。

### 2.5 代理模式扩展

在前面介绍的代理模式中，代理类中包含了对真实主题的引用，这种方式存在两个缺点。

1. 真实主题与代理主题一一对应，增加真实主题也要增加代理。
2. 设计代理以前真实主题必须事先存在，不太灵活。采用动态代理模式可以解决以上问题，如 [Spring](http://c.biancheng.net/spring/)AOP，其结构图如图 4 所示。

![](./imgs/java_proxy_design_exetension.gif)

## 3、静态代理



## 4、JDK  动态代理

### 4.1 简介

静态代理特点是代理类和目标类在代码中是确定的，因此称为静态。静态代理可以在不修改目标对象功能的前提下，对目标功能进行扩展，但是静态代理不够灵活。动态代理也叫 JDK 代理或接口代理，有以下特点：

- 代理对象不需要实现接口
- 代理对象的生成是利用 JDK 的 API 动态的在内存中构建代理对象
- 能在代码运行时动态地改变某个对象的代理，并且能为代理对象动态地增加方法、增加行为

### 4.2实现

一般情况下，动态代理的底层不用我们亲自去实现，可以使用线程提供的 API 。例如，在 Java 生态中，目前普遍使用的是 JDK 自带的代理和 GGLib 提供的类库。

JDK 实现代理只需要使用 newProxyInstance 方法，该方法需要接收三个参数，语法格式如下：

`static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h )`

注意该方法在 Proxy 类中是静态方法，且接收的三个参数说明依次为：

- ClassLoader loader：指定当前目标对象使用类加载器，获取加载器的方法是固定的
- Class<?>[] interfaces：目标对象实现的接口的类型，使用泛型方式确认类型
- InvocationHandler h：事件处理，执行目标对象的方法时，会触发事件处理器的方法，把当前执行目标对象的方法作为参数传入

```Java
public class ProxyTest {
    public static void main(String[] args) {
        ISubject subject = new SubjectProxyDy().getInstance(new SubjectImpl());
        subject.request();
    }
}

// 抽象主题
interface ISubject {
    void request();
}

// 真实主题
class SubjectImpl implements ISubject {
    public void request() {
        System.out.println("访问真实主题方法...");
    }
}

// 动态代理
class SubjectProxyDy implements InvocationHandler {
    private ISubject mSubject;

    public ISubject getInstance(ISubject subject) {
        mSubject = subject;
        Class<?> clazz = subject.getClass();
        return (ISubject) Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        preRequest();
        Object result = method.invoke(this.mSubject, args);
        postRequest();
        return result;
    }

    public void preRequest() {
        System.out.println("访问真实主题之前的预处理。");
    }

    public void postRequest() {
        System.out.println("访问真实主题之后的后续处理。");
    }
}
```

### 4.3 静态代理和动态代理的区别

- 静态代理只能通过手动完成代理操作，如果被代理类增加了新的方法，则代理类需要同步增加，违背开闭原则。
- 动态代理采用在运行时动态生成代码的方式，取消了对被代理类的扩展限制，遵循开闭原则。
- 若动态代理要对目标类的增强逻辑进行扩展，结合策略模式，只需要新增策略类便可完成，无需修改代理类的代码

实现上的区别：

​    JDK静态代理是通过直接编码创建的，而JDK动态代理是利用反射机制在运行时创建代理类的。在动态代理中，核心是` InvocationHandler` 。每一个代理的实例都会有一个关联的调用处理程序(`InvocationHandler`)。对待代理实例进行调用时，将对方法的调用进行编码并指派到它的调用处理器(`InvocationHandler`)的`invoke`方法。所以对代理对象实例方法的调用都是通过`InvocationHandler`中的`invoke`方法来完成的，而`invoke`方法会根据传入的代理对象、方法名称以及参数决定调用代理的哪个方法。

## 4、CGLIB 动态代理 todo

// todo `Cglib代理(Code Generation Library)`

 https://www.jianshu.com/p/9a61af393e41

| 代理方式      | 实现                                                         | 优点                                                         | 缺点                                                         | 特点                                                       |
| :------------ | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :--------------------------------------------------------- |
| JDK静态代理   | 代理类与委托类实现同一接口，并且在代理类中需要硬编码接口     | 实现简单，容易理解                                           | 代理类需要硬编码接口，在实际应用中可能会导致重复编码，浪费存储空间并且效率很低 |                                                            |
| JDK动态代理   | 代理类与委托类实现同一接口，主要是通过代理类实现InvocationHandler并重写invoke方法来进行动态代理的，在invoke方法中将对方法进行增强处理 | 不需要硬编码接口，代码复用率高                               | 只能够代理实现了接口的委托类                                 | 底层使用反射机制进行方法的调用                             |
| CGLIB动态代理 | 代理类将委托类作为自己的父类并为其中的非final委托方法创建两个方法，一个是与委托方法签名相同的方法，它在方法中会通过super调用委托方法；另一个是代理类独有的方法。在代理方法中，它会判断是否存在实现了MethodInterceptor接口的对象，若存在则将调用intercept方法对委托方法进行代理 | 可以在运行时对类或者是接口进行增强操作，且委托类无需实现接口 | 不能对final类以及final方法进行代理                           | 底层将方法全部存入一个数组中，通过数组索引直接进行方法调用 |

## 5、Proxy.newProxyInstance 源码 todo

参考： https://www.jianshu.com/p/471c80a7e831

```Java
 public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException {
     //如果h为空将抛出异常
     Objects.requireNonNull(h);

     final Class<?>[] intfs = interfaces.clone();//拷贝被代理类实现的一些接口，用于后面权限方面的一些检查
     final SecurityManager sm = System.getSecurityManager();
     if (sm != null) {
         //在这里对某些安全权限进行检查，确保我们有权限对预期的被代理类进行代理
         checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
     }

     /*
         * 下面这个方法将产生代理类
         */
     Class<?> cl = getProxyClass0(loader, intfs);

     /*
         * 使用指定的调用处理程序获取代理类的构造函数对象
         */
     try {
         if (sm != null) {
             checkNewProxyPermission(Reflection.getCallerClass(), cl);
         }

         final Constructor<?> cons = cl.getConstructor(constructorParams);
         final InvocationHandler ih = h;
         //假如代理类的构造函数是private的，就使用反射来set accessible
         if (!Modifier.isPublic(cl.getModifiers())) {
             AccessController.doPrivileged(new PrivilegedAction<Void>() {
                 public Void run() {
                     cons.setAccessible(true);
                     return null;
                 }
             });
         }
         //根据代理类的构造函数来生成代理类的对象并返回
         return cons.newInstance(new Object[]{h});
     } catch (IllegalAccessException|InstantiationException e) {
         throw new InternalError(e.toString(), e);
     } catch (InvocationTargetException e) {
         Throwable t = e.getCause();
         if (t instanceof RuntimeException) {
             throw (RuntimeException) t;
         } else {
             throw new InternalError(t.toString(), t);
         }
     } catch (NoSuchMethodException e) {
         throw new InternalError(e.toString(), e);
     }
 }
```



参考：

1、http://c.biancheng.net/view/1359.html