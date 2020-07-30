---

layout: post
title: "[Java关键字]synchronized关键字的使用"
date: 2020-07-29
tag: Java关键字


---





> **synchronized可以保证方法或者代码块在运行时，同一时刻只有一个方法可以进入到临界区，同时它还可以保证共享变量的内存可见性**



### synchronized的作用

`synchronized`解决线程安全问题。

基本上所有的并发模式在解决线程安全问题时，都采用“序列化访问临界资源”的方案，即在同一时刻，只能有一个线程访问临界资源，也称作同步互斥访问。

通常来说，是在访问临界资源的代码前面加上一个锁，当访问完临界资源后释放锁，让其他线程继续访问。

### synchronized用法

1、修饰一个代码块，作用的对象是调用这个代码块的对象

2、修饰一个方法，作用的对象是调用这个方法的对象

3、修饰一个静态方法，作用的对象是静态方法所属的类的所有对象

4、修饰一个类，作用的对象是该类的所有对象。

### synchronized修饰一个代码块

#### *<u>1.1 一个线程访问一个对象obj中的synchronize(this)同步代码块时，其它线程试图访问该对象obj的synchronize(this)同步块时将会被阻塞。</u>*

```java
class MyThread implements Runnable{
        private static final int NUM = 3;
        @Override
        public void run() {
            synchronized(this){
                for(int i =0;i<NUM;i++){
                    System.out.println(Thread.currentThread().getName()+"  running .....");
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

            }
        }

    }

 public class SyncCodeBlock {

        public static void main(String[] args) {
            MyThread mt = new MyThread();
            new Thread(mt,"Thread1").start();
            new Thread(mt,"Thread2").start();
        }

    }
```

**运行结果：**

```java
Thread2  running ..... 
Thread2  running ..... 
Thread2  running ..... 
Thread1  running ..... 
Thread1  running ..... 
Thread1  running .....
```

这里，名字为Thread1、Thread2的两个线程都想访问对象mt的同步块synchronized(this),由于只有一个锁，谁拿到的这个锁就该谁访问，也就是说，在任一时刻只能有一个线程访问这一同步代码块。例如，当Thread1线程拿到锁正在执行run里面的代码时，名字为Thread2的线程也想访问同一对象mt的同步块synchroized(this)的同步块将会被阻塞，只有等Thread1访问结束之后释放该对象Thread2拿到该对象锁才能访问。



如果我们将测试代码该为如下：

```java
public static void main(String[] args){
        MyThread mt = new MyThread();
        MyThread mt2 = new MyThread();
        new Thread(mt,"Thread1").start();
        new Thread(mt2,"Thread2").start();
    }
```

**结果如下：**

```java
Thread2  running .....
Thread1  running .....
Thread2  running .....
Thread1  running .....
Thread2  running .....
Thread1  running .....
```

这里，和上面的第一种测试代码不同，这里有两个不同的MyThread对象，分别对应着两把锁，因此线程Thread1访问同步块是去拿对象mt的锁，而线程Thread2访问同步块是去拿对象mt2的锁，而这两个锁是没有任何关系的，即线程Thread1执行的是对象mt的中的synchronized代码块，而线程Thread2执行的是对象mt2中的synchronized代码块，两者互不相关，因此这两个线程就可以同时进行。

**小结：当一个线程访问一个对象obj中的synchronized(this)同步块时，其它线程访问这个对象obj的synchronize(this)就会阻塞**



#### *<u>1.2 当一个线程访问一个对象obj中的synchronized(this)同步代码块时，其它的线程可以同时访问该对象obj中的非synchronize(this)代码块</u>*

```java
class MyThread implements Runnable{
        private static final int NUM = 3;
        @Override
        public void run() {
            String name = Thread.currentThread().getName();
            if(name.equals("Thread1")){
                synMethod();
            }
            else if(name.equals("Thread2")){
                notSynMethod();
            }
        }
        public void synMethod(){
            synchronized(this){
                for(int i=0;i<NUM;i++){
                    System.out.println(Thread.currentThread().getName()+"running ....");
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
        public void notSynMethod(){
            for(int i=0;i<NUM;i++){
                System.out.println(Thread.currentThread().getName()+"running ....");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }
```

```java
 public static void main(String[] args) {
        MyThread mt = new MyThread();
        new Thread(mt,"Thread1").start();
        new Thread(mt,"Thread2").start();
    }
```

**运行结果：**

```java
Thread1 running ....
Thread2 running ....
Thread1 running ....
Thread2 running ....
Thread1 running ....
Thread2 running ....
```

上面的代码中，有一个使用synchronize(this)同步的方法synMethod，有一个没有使用关键字synchronize(this)同步的方法notSynMethod。

由于线程Thread1和Thread2没有同时访问对象mt的synchronized(this)同步代码块，而是只有Thread1访问，Thread2访问的是对象mt非synchronized(this)代码块。因此这两个线程是可以同时进行的。



看下面的例子，将上例Demo2中的notSynMethod方法用synchronized(otherRefe)来同步

```java
  private int [] value = new int[0];
    public void notSynMethod(){
        synchronized(value){
            for(int i=0;i<NUM;i++){
                System.out.println(Thread.currentThread().getName()+" running ....");
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }
```

改成这样，线程Thread1、Thead2依然可以同时工作，这是因为他们所需要的锁不一样。

**小结：当一个线程访问对象obj的synchronized(this)代码块时，其它线程是可以同时访问该对象obj的非synchronized(this)的代码块**



#### *<u>1.3 当多线程并发时，一个线程访问对象obj的synchronized(this)代码块时，其它线程对对象obj中所有其他synchronized(this)同步代码块的访问将被阻塞</u>*

```java
class MyThread5 implements Runnable{
        private static final int NUM = 3;
        @Override
        public void run() {
            String name = Thread.currentThread().getName();
            if(name.equals("Thread1")){
                synMethod();
            }
            else if(name.equals("Thread2")){
                synMethod2();
            }
        }
        public void synMethod(){
            synchronized(this){
                for(int i=0;i<NUM;i++){
                    System.out.println(Thread.currentThread().getName()+" running ....");
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
        public void synMethod2(){
            synchronized(this){
                for(int i=0;i<NUM;i++){
                    System.out.println(Thread.currentThread().getName()+" running ...." + i);
                    try {
                        Thread.sleep(10);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }

        }

    }
```

```java
public static void main(String[] args) {
        MyThread5 mt = new MyThread5();
        new Thread(mt,"Thread1").start();
        new Thread(mt,"Thread2").start();
    }
```

**运行结果：**

```java
Thread2 running ....0
Thread2 running ....1
Thread2 running ....2
Thread1 running ....
Thread1 running ....
Thread1 running ....
```

当线程Thread1访问对象mt的方法synMethod中的synchronized(this)同步代码块时，线程Thread2访问对象mt中另一个方法synMethod2中synchronized(this)同步代码块的访问将被阻塞。这是因为，线程Thread1访问mt的一个synchronized(this)同步代码块时，它就获得了这个mt的对象锁。结果，线程Thread2对该mt对象所有使用synchronized(this)的同步代码部分的访问都被暂时阻塞。



### synchronized修饰某个方法

synchronized修饰某个方法，作用的对象为调用该方法的对象。

```java
 public synchronized void method(){
        //to do something ....
    }
```

效果等同于：

```java
public void method(){
        synchronized(this){
            //to do something ....
        }
    }
```

因此，这里不再进行介绍。

不过，有两点需要注意的是

**1、一般我们不建议同步整个方法，能同步方法中的某个代码块就同步代码块。同步安全往往是以性能为代价的。**

**2、synchronized关键字同步的方法不能继承。虽然可以使用synchronized来修饰某个方法，但是synchronized并不属于方法定义的一部分，因此，synchronized关键字不能被继承。如果在父类中的某个方法使用了synchronized关键字，而在子类中覆盖了这个方法，在子类中的这个方法默认情况下并不是同步的，而必须显式地在子类的这个方法中加上synchronized关键字才可以。**



### synchronized修饰静态代码块

**synchronized关键字修饰静态代码块，作用的是该类的所用对象。**

```java
class MyThread2 implements Runnable{

        private static final int NUM = 3;
        @Override
        public void run() {
            print();
        }

        public synchronized static void print() {
            for(int i=0;i<NUM;i++){
                System.out.println(Thread.currentThread().getName()+" running... "+i);
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

    }
```

```java
public static void main(String[] args) {
        MyThread2 mt = new MyThread2();
        MyThread2 mt2 = new MyThread2();
        new Thread(mt,"Thread1").start();
        new Thread(mt2,"Thread2").start();

    }
```

**运行结果：**

```java
Thread1 running... 0
Thread1 running... 1
Thread1 running... 2
Thread2 running... 0
Thread2 running... 1
Thread2 running... 2
```

我们都知道静态方法是属于类的，不是属于对象的。

MyThread2类中的静态方法print使用了synchronized修饰，尽管在测试代码中使用了两个不同的对象mt/mt2,但是这里的锁不再是锁对象mt/mt2.而是锁”类MyThread2”这个对象，mt/mt2都是属性类MyThread的，因此mt/mt2就相当于同一把锁。，因此，这两个线程在访问此静态方法print时就需要取得锁之后才能访问，访问完之后释放锁。



### synchronized修饰一个类

```java
 class MyThread3 implements Runnable{

        private static final int NUM = 3;
        @Override
        public void run() {
            synchronized(MyThread3.class){
                for(int i=0;i<NUM;i++){
                    System.out.println(Thread.currentThread().getName()+" running... "+i);
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }

    }
```

```java
public static void main(String[] args) {
        MyThread3 mt = new MyThread3();
        MyThread3 mt2 = new MyThread3();
        new Thread(mt,"Thread1").start();
        new Thread(mt2,"Thread2").start();
    }
```

**运行结果：**

```java
Thread2 running... 0
Thread2 running... 1
Thread2 running... 2
Thread1 running... 0
Thread1 running... 1
Thread1 running... 2
```

**由于mt/mt2都是属于类MyThread3的，而synchronized修饰的是MyThread3这整个类，因此所有的MyThread访问此代码块都是互斥的，任一时刻都只能有一个线程能够访问。**

不知道大家有没有这样的疑问，反正我是有的，既然synchronized修饰的静态方法和修饰的整个类都是作用与该类的全部对象，那么这两者是不是互斥的呢？？

```java
class MyThread4 implements Runnable{

        private static final int NUM = 3;
        @Override
        public void run() {
            String name = Thread.currentThread().getName();
            if(name.equals("Thread1")){
                synClassMethod();
            }
            else if(name.equals("Thread2")){
                synStaticCodeBlock();
            }
        }

        public void synClassMethod(){
            synchronized(MyThread4.class){
                for(int i=0;i<NUM;i++){
                    System.out.println(Thread.currentThread().getName()+" running... "+i);
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
        public static void synStaticCodeBlock(){    
            for(int i=0;i<NUM;i++){
                System.out.println(Thread.currentThread().getName()+" running... "+i);
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }

    }
```

```java
public static void main(String[] args) {
        MyThread4 mt = new MyThread4();
        MyThread4 mt2 = new MyThread4();
        new Thread(mt,"Thread1").start();
        new Thread(mt2,"Thread2").start();
    }
```

**运行结果：**

```java
Thread2 running... 0
Thread1 running... 0
Thread2 running... 1
Thread1 running... 1
Thread2 running... 2
Thread1 running... 2
```

**虽然都作用与该类的所用对象，但是锁却不是同一个锁，因此不是互斥的，可以同时访问**。



### 总结

1. 当两个线程Thread1、Thread2访问一个对象obj的synchronized(this)代码块时，每个时刻都只能有一个访问此代码块，当Thread1访问时，Thread2只有在Thread1线程执行完这段代码块并释放锁后取得锁才能访问。
2. 当两个线程并发时，一个线程访问对象obj的synchronized(this)代码块时，其它的线程可以访问对象obj的非synchronized(this)代码块。
3. 当多线程并发时，一个线程访问对象obj的synchronized(this)代码块时，其它线程对对象obj中所有其他synchronized(this)同步代码块的访问将被阻塞。这是因为，当一个线程访问obj的一个synchronized(this)同步代码块时，它就获得了这个obj的对象锁。结果，其它线程对该obj对象所有同步代码部分的访问都被暂时阻塞。
4. 第3个结论同样适用其它同步代码块。例如，如果一个线程访问对象lock的synchronized(lock)的同步代码块，其中private int[] lock = new int[0]，则其它线程在在其它方法中出现的
   synchronized(lock)的同步代码块的访问将被阻塞。
5. 上面所有的结论都适用于其它对象锁。

