# Java_Try_Catch_Finally_Return

整理了 Java 异常捕获代码块，Finally 语句对返回值的影响。

<!--more-->

## 1、简介

## 2、表现

### 2.0 预定义

```java
static class RETURN_TYPE{
  public static int RETURN_TYPE_DEFAULT = 0;
  public static int RETURN_TYPE_TRY = 1;
  public static int RETURN_TYPE_CATCH = 2;
  public static int RETURN_TYPE_FINALLY = 3;
  public static int RETURN_TYPE_OUTER = 4;
}
```

### 2.1 try中有return，finally中没有return，没改变返回值

```java
public class TryCatchFinallyReturnMain {
  public static void main(String[] args) {
    System.out.println("\n----- testReturnCodeSequence start -----");
    int testSeqRes = testReturnCodeSequence();
    System.out.println("  testReturnCodeSequence Res:" + testSeqRes);
    System.out.println("----- testReturnCodeSequence end -----");
  }
  
  private static int testReturnCodeSequence() {
    int res = RETURN_TYPE.RETURN_TYPE_DEFAULT;
    try {
      res = RETURN_TYPE.RETURN_TYPE_TRY;
      System.out.println("testReturnCodeSequence try");
      return res;
    } catch (Exception ex) {
      System.out.println("testReturnCodeSequence catch");
    } finally {
      System.out.println("testReturnCodeSequence finally");
    }
    System.out.println("testReturnCodeSequence outer");
    res = RETURN_TYPE.RETURN_TYPE_OUTER;
    return res;
  }
}
```

结果，返回值是try块的：

```shell
----- testReturnCodeSequence start -----
testReturnCodeSequence try
testReturnCodeSequence finally
  testReturnCodeSequence Res:1
----- testReturnCodeSequence end -----
```

### 2.2 try和finally中均有return

```Java
public class TryCatchFinallyReturnMain {
  public static void main(String[] args) {
    System.out.println("\n----- testReturnWhenTryButChangeWhenFinally start -----");
    int testReturn1 = testReturnWhenTryButChangeWhenFinally();
    System.out.println("  testReturnWhenTryButChangeWhenFinally Res:" + testReturn1);
    System.out.println("----- testReturnWhenTryButChangeWhenFinally end -----");
  }

  private static int testReturnWhenTryButChangeWhenFinally() {
    int res = RETURN_TYPE.RETURN_TYPE_DEFAULT;
    try {
      res = RETURN_TYPE.RETURN_TYPE_TRY;
      System.out.println("testReturnWhenTryButChangeWhenFinally try");
      return res;
    } catch (Exception ex) {
      System.out.println("testReturnWhenTryButChangeWhenFinally catch");
      res = RETURN_TYPE.RETURN_TYPE_CATCH;
    } finally {
      System.out.println("testReturnWhenTryButChangeWhenFinally finally");
      res = RETURN_TYPE.RETURN_TYPE_FINALLY;
    }
    res = RETURN_TYPE.RETURN_TYPE_OUTER;
    System.out.println("testReturnWhenTryButChangeWhenFinally outer");
    return res;
  }
}
```

结果，返回值是try块的：

```shell
----- testReturnWhenTryButChangeWhenFinally start -----
testReturnWhenTryButChangeWhenFinally try
testReturnWhenTryButChangeWhenFinally finally
  testReturnWhenTryButChangeWhenFinally Res:1
----- testReturnWhenTryButChangeWhenFinally end -----
```

### 2.3 try中retrun, finally中改变值类型返回值

```java
public class TryCatchFinallyReturnMain {
  public static void main(String[] args) {
    System.out.println("\n----- testReturnWhenTryAndFinally start -----");
    int testReturn2 = testReturnWhenTryAndFinally();
    System.out.println("  testReturnWhenTryAndFinally Res:" + testReturn2);
    System.out.println("----- testReturnWhenTryAndFinally end -----");
  }
  private static int testReturnWhenTryAndFinally() {
    int res = RETURN_TYPE.RETURN_TYPE_DEFAULT;
    try {
      res = RETURN_TYPE.RETURN_TYPE_TRY;
      System.out.println("testReturnWhenTryAndFinally try");
      return res;
    } catch (Exception ex) {
      res = RETURN_TYPE.RETURN_TYPE_CATCH;
      System.out.println("testReturnWhenTryAndFinally catch");
    } finally {
      res = RETURN_TYPE.RETURN_TYPE_FINALLY;
      System.out.println("testReturnWhenTryAndFinally finally");
      return res;
    }
  }
}
```

结果，返回值是finally块的：

```java
----- testReturnWhenTryAndFinally start -----
testReturnWhenTryAndFinally try
testReturnWhenTryAndFinally finally
  testReturnWhenTryAndFinally Res:3
----- testReturnWhenTryAndFinally end -----
```

### 2.4 try中retrun, finally中改变引用类型返回值

```Java
public class TryCatchFinallyReturnMain {
  public static void main(String[] args) {
    System.out.println("\n----- testReturnRefWhenTryButChangeWhenFinally start -----");
    ReturnId testReturn3 = testReturnRefWhenTryButChangeWhenFinally();
    System.out.println("  testReturnRefWhenTryButChangeWhenFinally Res:" + testReturn3.id);
    System.out.println("----- testReturnRefWhenTryButChangeWhenFinally end -----");
  }

  private static ReturnId testReturnRefWhenTryButChangeWhenFinally() {
    ReturnId res = new ReturnId();
    try {
      res.id = RETURN_TYPE.RETURN_TYPE_TRY;
      System.out.println("testReturnRefWhenTryButChangeWhenFinally try");
      return res;
    } catch (Exception ex) {
      System.out.println("testReturnRefWhenTryButChangeWhenFinally catch");
      res.id = RETURN_TYPE.RETURN_TYPE_CATCH;
    } finally {
      System.out.println("testReturnRefWhenTryButChangeWhenFinally finally");
      res.id = RETURN_TYPE.RETURN_TYPE_FINALLY;
    }
    res.id = RETURN_TYPE.RETURN_TYPE_OUTER;
    System.out.println("testReturnRefWhenTryButChangeWhenFinally outer");
    return res;
  }

  static class ReturnId{
    public int id = RETURN_TYPE.RETURN_TYPE_DEFAULT;
  }
}
```

结果，返回值是finally块的：

```shell
----- testReturnRefWhenTryButChangeWhenFinally start -----
testReturnRefWhenTryButChangeWhenFinally try
testReturnRefWhenTryButChangeWhenFinally finally
  testReturnRefWhenTryButChangeWhenFinally Res:3
----- testReturnRefWhenTryButChangeWhenFinally end -----
```

### 2.5 总结

* 如果finally中没有return语句，也没有改变要返回值，则执行完finally中的语句后，会接着执行try/catch中的return语句，返回之前保留的值。
* 如果finally中有return语句，则会将try/catch中的return语句”覆盖“掉，直接执行finally中的return语句，得到返回值，这样便无法得到try/catch之前保留好的返回值。
* 如果finally中没有return语句，但是改变了要返回的值
    * 值传递：如果return的数据是基本数据类型或文本字符串，则在finally中对该基本数据的改变不起作用，try/catch中的return语句依然会返回进入finally块之前保留的值。
    * 引用传递：如果return的数据是引用数据类型，而在finally中对该引用数据类型的属性值的改变起作用，try/catch中的return语句返回的就是在finally中改变后的该属性的值。
* finally中最好不要包含return，否则程序会提前退出，返回值不是try或catch中保存的返回值。编译器把finally中的return实现为一个warning。

## 3、原理

当通过压栈传递参数时，参数的类型不同，压栈的内容也不同。如果是值类型，压栈的就是经过复制的参数值，如果是引用类型，那么进栈的只是一个引用，这也就是我们所熟悉的，传递值类型时，函数内修改参数值不会影响函数外，而引用类型的话则会影响。