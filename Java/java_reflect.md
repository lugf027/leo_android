

# Java_Reflect

## 1、简介

反射 (Reflection) 是 Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。

通过反射机制，可以在运行期间访问 Java 对象的属性，方法，构造方法等。

* 应用场景：
    * 动态代理
    * 注解
    * 开发通用框架
    * 可扩展性功能
* 缺点
    * 性能开销
    * 破坏封装性
    * 内部曝光

## 2、反射机制原理

### 2、1 类加载过程

1. 在编译时，Java 编译器编译好 `.java` 文件之后，在磁盘中产生 `.class` 文件。`.class` 文件是二进制文件，内容是只有 JVM 能够识别的机器码。
2. JVM 中的类加载器读取字节码文件，取出二进制数据，加载到内存中，解析.class 文件内的信息。类加载器会根据类的全限定名来获取此类的二进制字节流；然后，将字节流所代表的静态存储结构转化为方法区的运行时数据结构；接着，在内存中生成代表这个类的 `java.lang.Class` 对象。
3. 加载结束后，JVM 开始进行连接阶段（包含验证、准备、初始化）。经过这一系列操作，类的变量会被初始化。

### 2.2 Class 类

​    除了`int`等基本类型外，Java的其他类型全部都是`class`（包括`interface`）。而`class`是由JVM在执行过程中动态加载的。JVM在第一次读取到一种`class`类型时，将其加载进内存。

​    每加载一种`class`，JVM就为其创建一个`Class`类型的实例，并关联起来。注意：这里的`Class`类型是一个名叫`Class`的`class`。

```java
package java.lang;

public final class Class {
    private Class() {}
}
```

​    以`String`类为例，当JVM加载`String`类时，它首先读取`String.class`文件到内存，然后，为`String`类创建一个`Class`实例并关联起来：

```Java
Class cls = new Class(String);
```

这个`Class`实例是JVM内部创建的，且Class`类的构造方法是`private`，只有JVM能创建`Class`实例，我们自己的Java程序是无法创建`Class`实例的。

所以，JVM持有的每个`Class`实例都指向一个数据类型（`class`或`interface`）：

```asciiarmor
┌───────────────────────────┐
│      Class Instance       │──────> String
├───────────────────────────┤
│name = "java.lang.String"  │
└───────────────────────────┘
┌───────────────────────────┐
│      Class Instance       │──────> Random
├───────────────────────────┤
│name = "java.util.Random"  │
└───────────────────────────┘
```

一个`Class`实例包含了该`class`的所有完整信息：

```asciiarmor
┌───────────────────────────┐
│      Class Instance       │──────> String
├───────────────────────────┤
│name = "java.lang.String"  │
├───────────────────────────┤
│package = "java.lang"      │
├───────────────────────────┤
│super = "java.lang.Object" │
├───────────────────────────┤
│interface = CharSequence...│
├───────────────────────────┤
│field = value[],hash,...   │
├───────────────────────────┤
│method = indexOf()...      │
└───────────────────────────┘
```

由于JVM为每个加载的`class`创建了对应的`Class`实例，并在实例中保存了该`class`的所有信息，包括类名、包名、父类、实现的接口、所有方法、字段等，因此，如果获取了某个`Class`实例，我们就可以通过这个`Class`实例获取到该实例对应的`class`的所有信息。

由此，反射可以使用起来了。

### 2.3 方法的反射调用

`Method.invoke` 方法实际上委派给 `MethodAccessor` 接口来处理。它有两个已有的具体实现：

- `NativeMethodAccessorImpl`：本地方法来实现反射调用
- `DelegatingMethodAccessorImpl`：委派模式来实现反射调用

测试方法与调用路径

```Java
package com.company;

import java.lang.reflect.Method;

public class Main {

    public static void target(int i) {
        new Exception("#" + i).printStackTrace();
    }

    public static void main(String[] args) throws Exception {
        Class<?> klass = Class.forName("com.company.Main");
        Method method = klass.getMethod("target", int.class);
        for (int i = 0; i < 20; i++) {
            method.invoke(null, i);
        }
    }
}

/* 堆栈
...
java.lang.Exception: #15
	at com.company.Main.target(Main.java:8)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.company.Main.main(Main.java:15)
java.lang.Exception: #16
	at com.company.Main.target(Main.java:8)
	at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at com.company.Main.main(Main.java:15)
...*/
```

​    每个 `Method` 实例的第一次反射调用都会生成一个委派实现（`DelegatingMethodAccessorImpl`），它所委派的具体实现便是一个本地实现（`NativeMethodAccessorImpl`）。本地实现非常容易理解。当进入了 Java 虚拟机内部之后，我们便拥有了 `Method` 实例所指向方法的具体地址。这时候，反射调用无非就是将传入的参数准备好，然后调用进入目标方法。

​    为什么反射调用`DelegatingMethodAccessorImpl` ，采取委托实现作为中间层，而不是直接交给本地实现？

​    其实，Java 的反射调用机制还设立了另一种动态生成字节码的实现（下称动态实现），直接使用 invoke 指令来调用目标方法。之所以采用委派实现，便是为了能够在本地实现以及动态实现中切换。动态实现和本地实现相比，其运行效率要快上 20 倍。这是因为动态实现无需经过 Java 到 C++ 再到 Java 的切换，但由于生成字节码十分耗时，仅调用一次的话，反而是本地实现要快上 3 到 4 倍。

​    考虑到许多反射调用仅会执行一次，Java 虚拟机设置了一个阈值 15（可以通过 `-Dsun.reflect.inflationThreshold` 来调整），当某个反射调用的调用次数在 15 之下时，采用本地实现；当达到 15 时，便开始动态生成字节码，并将委派实现的委派对象切换至动态实现，这个过程我们称之为 Inflation机制。

​    反射调用的 inflation 机制是可以通过参数(-Dsun.reflect.noinflation = true)来关闭的。这样一来，在反射调用一开始便会直接生成动态实现，而不会使用委派实现或者本地实现。

### 2.4 反射调用的开销

主要性能开销：

* 接口的通用性——变长参数方法导致 invoke 方法是传 object 和 object[] 数组的

> Method.invoke 是一个变长参数方法，在字节码层面它的最后一个参数会是 Object 数组（感兴趣的同学私下可以用 javap 查看）。Java 编译器会在方法调用处生成一个长度为传入参数数量的 Object 数组，并将传入参数一一存储进该数组中。
>
> 除了带来性能开销外，还可能占用堆内存，使得 GC 更加频繁。

* 基本参数类型需要装箱和拆箱

> 由于 Object 数组不能存储基本类型，Java 编译器会对传入的基本类型参数进行自动装箱。
>
> 除了产生大量额外的对象和内存开销，带来性能开销外，还可能占用堆内存，使得 GC 更加频繁。

* 编译器难以对动态调用的代码提前做优化比如方法内联

* 反射需要按名检索类和方法，有一定的时间开销

> 注意，以 `getMethod` 为代表的查找方法操作，会返回查找得到结果的一份拷贝。因此，我们应当避免在热点代码中使用返回 `Method` 数组的 `getMethods` 或者 `getDeclaredMethods` 方法，以减少不必要的堆空间消耗。
>
> 在实践中，我们往往会在应用程序中缓存 `Class.forName` 和 `Class.getMethod` 的结果。



## 3、使用方法

###  3.1 java.lang.reflect 包

Java 中的 `java.lang.reflect` 包提供了反射功能。`java.lang.reflect` 包中的类都没有 `public` 构造方法。

`java.lang.reflect` 包的核心接口和类如下：

- `Member` 接口：反映关于单个成员(字段或方法)或构造函数的标识信息。
- `Field` 类：提供一个类的域的信息以及访问类的域的接口。
- `Method` 类：提供一个类的方法的信息以及访问类的方法的接口。
- `Constructor` 类：提供一个类的构造函数的信息以及访问类的构造函数的接口。
- `Array` 类：该类提供动态地生成和访问 JAVA 数组的方法。
- `Modifier` 类：提供了 static 方法和常量，对类和成员访问修饰符进行解码。
- `Proxy` 类：提供动态地生成代理类和类实例的静态方法。

#### 3.2 获取一个`class`的`Class`实例

* 直接通过一个`class`的静态变量`class`获取

```java
Class cls = String.class;
```

* 通过实例变量提供的`getClass()`方法获取

```Java
String s = "Hello";
Class cls = s.getClass();
```

* 经由一个`class`的完整类名，通过静态方法`Class.forName()`获取

```java
Class cls = Class.forName("java.lang.String");
```

#### 3.3 创建实例

- 用 `Class` 对象的 `newInstance` 方法（局限：只能调用该类的public无参数构造方法）。

```Java
Person p = Person.class.newInstance();
```

- 用 `Constructor` 对象的 `newInstance` 方法。

​    为了调用任意的构造方法，Java的反射API提供了Constructor对象，它包含一个构造方法的所有信息，可以创建一个实例。Constructor对象和Method非常类似，不同之处仅在于它是一个构造方法，并且，调用结果总是返回实例

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 获取构造方法Integer(int):
        Constructor cons1 = Integer.class.getConstructor(int.class);
        // 调用构造方法:
        Integer n1 = (Integer) cons1.newInstance(123);
        System.out.println(n1);
        // 获取构造方法Integer(String)
        Constructor cons2 = Integer.class.getConstructor(String.class);
        Integer n2 = (Integer) cons2.newInstance("456");
        System.out.println(n2);
    }
}
```
​    通过Class实例获取Constructor的方法如下:

- `getConstructor(Class...)`：获取某个`public`的`Constructor`；
- `getDeclaredConstructor(Class...)`：获取某个`Constructor`；
- `getConstructors()`：获取所有`public`的`Constructor`；
- `getDeclaredConstructors()`：获取所有`Constructor`。

​    调用非`public`的`Constructor`时，必须首先通过`setAccessible(true)`设置允许访问。`setAccessible(true)`可能会失败。

#### 3.4 创建数组实例

​    数组在 Java 里是比较特殊的一种类型，它可以赋值给一个对象引用。Java 中，**通过 `Array.newInstance` 创建数组的实例**。

```java
public class ReflectArrayDemo {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> cls = Class.forName("java.lang.String");
        Object array = Array.newInstance(cls, 25);
        //往数组里添加内容
        Array.set(array, 0, "Scala");
        Array.set(array, 1, "Java");
        Array.set(array, 2, "Groovy");
        Array.set(array, 3, "Scala");
        Array.set(array, 4, "Clojure");
        //获取某一项的内容
        System.out.println(Array.get(array, 3));
    }
}
//Output:
//Scala
```

#### 3.5 Field

​    `Class`类提供了以下几个方法来获取`Field`：

- Field getField(name)：根据字段名获取某个public的field（包括父类）
- Field getDeclaredField(name)：根据字段名获取当前类的某个field（不包括父类）
- Field[] getFields()：获取所有public的field（包括父类）
- Field[] getDeclaredFields()：获取当前类的所有field（不包括父类）

​    一个`Field`对象包含了一个字段的所有信息：

- `getName()`：返回字段名称，例如，`"name"`；
- `getType()`：返回字段类型，也是一个`Class`实例，例如，`String.class`；
- `getModifiers()`：返回字段的修饰符，它是一个`int`，不同的bit表示不同的含义。

​    设置字段值，`Field.set(Object, Object)`实现的，其中第一个`Object`参数是指定的实例，第二个`Object`参数是待修改的值。

​    非`public`修饰的字段直接访问会 IllegalAccessException，调用`Field.setAccessible(true)`的设置允许访问。

#### 3.6 Method

`Class`类提供了以下几个方法来获取`Method`：

- `Method getMethod(name, Class...)`：获取某个`public`的`Method`（包括父类）
- `Method getDeclaredMethod(name, Class...)`：获取当前类的某个`Method`（不包括父类）
- `Method[] getMethods()`：获取所有`public`的`Method`（包括父类）
- `Method[] getDeclaredMethods()`：获取当前类的所有`Method`（不包括父类）

一个`Method`对象包含一个方法的所有信息：

- `getName()`：返回方法名称，例如：`"getScore"`；
- `getReturnType()`：返回方法返回值类型，也是一个Class实例，例如：`String.class`；
- `getParameterTypes()`：返回方法的参数类型，是一个Class数组，例如：`{String.class, int.class}`；
- `getModifiers()`：返回方法的修饰符，它是一个`int`，不同的bit表示不同的含义。

​    对`Method`实例调用`invoke`就相当于调用该方法（`Object invoke(Object instance, Object... parameters)`），`invoke`的第一个参数是对象实例，即在哪个实例上调用该方法，后面的可变参数要与方法参数一致，否则将报错。调用静态方法时，无需指定实例对象，`invoke`方法传入的第一个参数为`null`。通过设置`setAccessible(true)`来访问非`public`方法。

## 4、补充

### 4.1、`Class`实例比较和`instanceof`的差别

因为`Class`实例在JVM中是唯一的，所以，上述方法获取的`Class`实例是同一个实例。可以用`==`比较两个`Class`实例。这种比较跟`instanceof`比较，是有差异的。

```java
Integer n = new Integer(123);

boolean b1 = n instanceof Integer; // true，因为n是Integer类型
boolean b2 = n instanceof Number; // true，因为n是Number类型的子类

Class classN = n.getClass();
boolean b3 = classN == Integer.class; // true，因为n.getClass()返回Integer.class
boolean b4 = classN == Number.class; // false，因为Integer.class!=Number.class
// boolean b4 = n.getClass() == Number.class 
// java: 不可比较的类型: java.lang.Class<capture#1, 共 ? extends java.lang.Integer>和java.lang.Class<java.lang.Number>

boolean b5 = Integer.class.isInstance(n); // true, 一个 Native 方法
boolean b6 = Number.class.isInstance(n); // true, 一个 Native 方法
```

### 4.2、举例JVM动态加载`class`

JVM总是动态加载`class`，可以在运行期根据条件来控制加载class。

### 4.3、`setAccessible(true)`可能会失败

​    如果JVM运行期存在`SecurityManager`，那么它会根据规则进行检查，有可能阻止`setAccessible(true)`。例如，某个`SecurityManager`可能不允许对`java`和`javax`开头的`package`的类调用`setAccessible(true)`，这样可以保证JVM核心库的安全。

### 4.4 一般情况下，反射调用耗时与次数关系

​    当某个反射调用的调用次数在 15 之下时，采用本地实现；当达到 15 时，便开始动态生成字节码，第15次比较耗时。之后的调用会更快。



// todo java 代理

参考：

1、https://www.liaoxuefeng.com/wiki/1252599548343744/1255945147512512

2、https://dunwu.github.io/javacore/basics/java-reflection.htm

3、https://zhuanlan.zhihu.com/p/55409796