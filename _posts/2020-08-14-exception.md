---


layout: post
title: "[Java基础]Java异常"
date: 2020-08-14
tag: Java基础


---

### 异常

在Java中，异常就是Java在编译、运行或运行过程中出现的错误。

程序错误分为三种：编译错误、运行时错误和逻辑错误

- 编译错误是因为程序没有遵循语法规则，编译程序能够自己发现并且提示我们错误的原因和位置，这个也是新手在刚接触编程语言时经常遇到的问题。
- 运行时错误是因为程序在执行时，运行环境发现了不能执行的操作。
- 逻辑错误是因为程序没有按照预期的逻辑顺序执行。异常也就是指程序运行时发生错误，而异常处理就是对这些错误进行处理和控制。



### **异常体系结构**

 * java.lang.Throwable

   |-----java.lang.Error:一般不编写针对性的代码进行处理。

   |-----java.lang.Exception:可以进行异常的处理
 * |------编译时异常(checked)

   |-----IOException

   |-----FileNotFoundException

   |-----ClassNotFoundException
 * |------运行时异常(unchecked,RuntimeException)

   |-----NullPointerException

   |-----ArrayIndexOutOfBoundsException

   |-----ClassCastException

   |-----NumberFormatException

   |-----InputMismatchException

   |-----ArithmeticException



### 异常的处理

#### 抓抛模型

1. "抛"：程序在正常执行的过程中，一旦出现异常，就会在异常代码处生成一个对应异常类的对象。 并将此对象抛出。  一旦抛出对象以后，其后的代码就不再执行。
2. "抓"：可以理解为异常的处理方式：① try-catch-finally  ② throws



### try-catch-finally的使用

```java
try{
		//可能出现异常的代码

}catch(异常类型1 变量名1){
		//处理异常的方式1
}catch(异常类型2 变量名2){
		//处理异常的方式2
 }catch(异常类型3 变量名3){
 		//处理异常的方式3
 }
 ....
 finally{
 		//一定会执行的代码
 }
```

####  说明：

 1. finally是可选的。
  2. 使用try将可能出现异常代码包装起来，在执行过程中，一旦出现异常，就会生成一个对应异常类的对象，根据此对象
    的类型，去catch中进行匹配
  3. 一旦try中的异常对象匹配到某一个catch时，就进入catch中进行异常的处理。一旦处理完成，就跳出当前的
    try-catch结构（在没有写finally的情况）。继续执行其后的代码
  4. catch中的异常类型如果没有子父类关系，则谁声明在上，谁声明在下无所谓。
    catch中的异常类型如果满足子父类关系，则要求子类一定声明在父类的上面。否则，报错
  5. 常用的异常对象处理的方式： ① String  getMessage()    ② printStackTrace()
  6. 在try结构中声明的变量，再出了try结构以后，就不能再被调用
  7. try-catch-finally结构可以嵌套

> - 使用try-catch-finally处理编译时异常，是得程序在编译时就不再报错，但是运行时仍可能报错。相当于我们使用try-catch-finally将一个编译时可能出现的异常，延迟到运行时出现。
> - 开发中，由于运行时异常比较常见，所以我们通常就不针对运行时异常编写try-catch-finally了。针对于编译时异常，我们说一定要考虑异常的处理。



####  try-catch-finally中finally的使用：

-  finally是可选的
- finally中声明的是一定会被执行的代码。即使catch中又出现异常了，try中有return语句，catch中有
   return语句等情况。
- 像数据库连接、输入输出流、网络编程Socket等资源，JVM是不能自动的回收的，我们需要自己手动的进行资源的
     释放。此时的资源释放，就需要声明在finally中。

```java
public static void main(String[] args) {
    int method = method();
    System.out.println(method);
}


public static int method(){

    try{
        int[] arr = new int[10];
        System.out.println(arr[10]);
        return 1;
    }catch(ArrayIndexOutOfBoundsException e){
        e.printStackTrace();
        return 2;
    }finally{
        System.out.println("我一定会被执行");
        return 3;
    }
}
```

```java
java.lang.ArrayIndexOutOfBoundsException: 10
	at com.heli.pdis.redisMsg.RedisPublish.method(RedisPublish.java:345)
	at com.heli.pdis.redisMsg.RedisPublish.main(RedisPublish.java:336)
我一定会被执行
3
```



### throws + 异常类型

- "throws + 异常类型"写在方法的声明处。指明此方法执行时，可能会抛出的异常类型。一旦当方法体执行时，出现异常，仍会在异常代码处生成一个异常类的对象，此对象满足throws后异常类型时，就会被抛出。异常代码后续的代码，就不再执行！

- 体会：try-catch-finally:真正的将异常给处理掉了。 throws的方式只是将异常抛给了方法的调用者。  并没有真正将异常处理掉。  

- 开发中如何选择使用try-catch-finally 还是使用throws？

     > -------如果父类中被重写的方法没有throws方式处理异常，则子类重写的方法也不能使用throws，意味着如果子类重写的方法中有异常，必须使用try-catch-finally方式处理。
     > -------执行的方法a中，先后又调用了另外的几个方法，这几个方法是递进关系执行的。我们建议这几个方法使用throws的方式进行处理。而执行的方法a可以考虑使用try-catch-finally方式进行处理。



### **异常场景汇总**

1、接口方法可以throws异常，但必须throws一个具体的异常，不能直接throws出去Exception

```java
public interface InterfaceException{
    void ExceptionMethod() throws NullPointerException;
}
```

2、接口方法throws异常，其实现类实现该方法的时候不强制必须抛出该异常，也可以任意抛出异常

```java
public class InterfaceExceptionImpl implements InterfaceException{
    public void ExceptionMethod(){
        
    }
}
```

```java
public class InterfaceExceptionImpl implements InterfaceException{
    public void ExceptionMethod() throws ArrayIndexOutOfBoundsException{
        
    }
}
```

3、catch块内如果捕获到了异常并且throw出去了e，那么方法之后的代码都不会再运行了；catch块内如果捕获到了异常，但是没有throw出去e，也没有任何导致程序终止的语句，那么try...catch...finally之后的语句仍然可以继续运行

4、捕获异常不可以先catch (Exception e){...}再catch (NullPointerException e)

原因是"Unreachable catch block for NullPointerException. It is already handled by the catch block for Exception*"，*即Java检测到NullPointer这个异常是不可达的

5、方法A声明throws异常，则调用方法A的代码必须try...catch...该异常

6、方法A声明throws出Exception，调用方法A的代码必须try...catch...该异常，如果：

（1）有匹配异常的catch块，则优先走匹配异常的catch块

（2）如果没有匹配异常的catch块，但是有catch (Exception e){...}，则走catch (Exception e){...}

（3）如果没有没有匹配的catch块，则调用方法A的地方throw异常，方法终止

7、如果方法中没有try...catch异常而该方法发生了异常，且方法声明中有throws，那么发生的该异常可以被抛到调用方法的代码中

8、如果方法中对某个代码块做了try...catch并且想把catch到的异常抛给调用方法的地方，那么：

（1）不可以只有throw没有throws，这将会导致捕获到的异常无法被抛给调用方法的地方

（2）不可以只有throws没有throw，这将会导致编译出错

只有在catch块中throw捕获到的异常并且在方法声明的地方throws异常，才可以将一个异常正确地抛给调用方法的地方