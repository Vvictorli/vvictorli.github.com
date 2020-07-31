---


layout: post
title: "[并发编程]ThreadLocal的作用及使用"
date: 2020-07-31
tag: 并发编程


---



> 
>
> 
>
> 
>
> 
>
> **ThreadLocal不是用来解决共享对象的多线程访问问题的**
>
> 
>
> 
>
> 

### **ThreadLocal的作用**

ThreadLocal通过ThreadLocal的set()方法设置到线程的ThreadLocal.ThreadLocalMap里的是是线程自己要存储的对象，其他线程不需要去访问，也是访问不到的。各个线程中的ThreadLocal.ThreadLocalMap以及ThreadLocal.ThreadLocal中的值都是不同的对象。

至于为什么要使用ThreadLocal，不妨这么考虑这个问题。Java Web中，写一个Servlet：

```java
public class Servlet extends HttpServlet
{

    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException
    {
        this.doGet(request, response);
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException
    {
        
    }
}
```

我在一个普通JavaBean内想拿到这个HttpServletRequest，但是无法通过参数传递的方式：

```java
public class OperateRequest
{
    public String operateRequest()
    {
        return null;
    }
}
```

这时候怎么办？第一个解决方案，Servlet类中定义一个全局的HttpServletRequest，至于怎么定义就随便了，可以定义成静态的，也可以定义成非静态的但是对外提供setter/getter，然后operateRequest()方法每次都取这个全局的HttpServletRequest就可以了。

不否认，这是一种可行的解决方案，但是这种解决方案有一个很大的缺点：竞争。既然HttpServletRequest是全局的，那势必要引入同步机制来保证线程安全性，引入同步机制意味着牺牲响应给用户的时间----这在注重与用户之间响应的Java Web中是难以容忍的。

所以，我们引入ThreadLocal，既然ThreadLocal.ThreadLocalMap是线程独有的，别的线程访问不了也没必要访问，那我们通过ThreadLocal把HttpServletRequest设置到线程的ThreadLocal.ThreadLocalMap里面去不就好了？这样，在一次请求中哪里需要用到HttpServletRequest，就使用ThreadLocal的get()方法就把这个HttpServletRequest给取出来了，是不是一个很好的解决方案呢？

### **ThreadLocal使用**

讨论 ThreadLocal 用在什么地方前，我们先明确下，如果仅仅就一个线程，那么都不用谈 ThreadLocal 的，**ThreadLocal 是用在多线程的场景的！！！**

ThreadLocal 归纳下来就 2 类用途：

- **保存线程上下文信息，在任意需要的地方可以获取！！！**
- **线程安全的，避免某些情况需要考虑线程安全必须同步带来的性能损失！！！**

#### 保存线程上下文信息，在任意需要的地方可以获取！！！

由于 ThreadLocal 的特性，同一线程在某地方进行设置，在随后的任意地方都可以获取到。从而可以用来保存线程上下文信息。

常用的比如每个请求怎么把一串后续关联起来，就可以用 ThreadLocal 进行 set，在后续的任意需要记录日志的方法里面进行 get 获取到请求 id，从而把整个请求串起来。

还有比如 Spring 的事务管理，用 ThreadLocal 存储 Connection，从而各个 DAO 可以获取同一 Connection，可以进行事务回滚，提交等操作。

> **备注：** ThreadLocal 的这种用处，很多时候是用在一些优秀的框架里面的，一般我们很少接触，反而下面的场景我们接触的更多一些！

#### 线程安全的，避免某些情况需要考虑线程安全必须同步带来的性能损失！！！

ThreadLocal 为解决多线程程序的并发问题提供了一种新的思路。但是 ThreadLocal 也有局限性，我们来看看阿里规范：

> 【参考】ThreadLocal 无法解决共享对象的更新问题，ThreadLocal 对象建议使用 static
>
> 修饰。这个变量是针对一个线程内所有操作共享的，所以设置为静态变量，所有此类实例共享 此静态变量 ，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只 要是这个线程内定义的)都可以操控这个变量。

### ThreadLocal 的最佳实践

由于线程的生命周期很长，如果我们往 ThreadLocal 里面 set 了很大很大的 Object 对象，虽然 set、get 等等方法在特定的条件会调用进行额外的清理，但是**ThreadLocal 被垃圾回收后，在 ThreadLocalMap 里对应的 Entry 的键值会变成 null，但是后续在也没有操作 set、get 等方法了。**

**所以最佳实践，应该在我们不使用的时候，主动调用 remove 方法进行清理。**

**最佳实践做法应该为：**

```java
try {
    // 其它业务逻辑
} finally {
    threadLocal 对象.remove();
}
```



### **ThreadLocal总结**

1、**ThreadLocal不是集合**，它不存储任何内容，真正存储数据的集合在Thread中。**ThreadLocal只是一个工具，一个往各个线程的ThreadLocal.ThreadLocalMap中table的某一位置set一个值的工具而已**

2、同步与ThreadLocal是解决多线程中数据访问问题的两种思路，**前者是数据共享的思路**，**后者是数据隔离的思路**

3、同步是一种以时间换空间的思想，ThreadLocal是一种空间换时间的思想

4、ThreadLocal既然是与线程相关的，那么对于Java Web来讲，ThreadLocal设置的值只在一次请求中有效，是不是和request很像？因为request里面的内容也只在一次请求有效，对比一下二者的区别：

（1）ThreadLocal只能存一个值，一个Request由于是Map形式的，可以用key-value形式存多个值

（2）ThreadLocal一般用在框架，Request一般用在表示层、Action、Servlet

