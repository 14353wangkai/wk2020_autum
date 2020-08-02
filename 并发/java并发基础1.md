## 两个原则

- happen before
  - 如果线程之间的变量有依赖性，那么两者的变量相互之间要具有可见性, 前者
- As-if-serial
  - 无论多线程中间怎么调度，和执行，最终结果要和单线程结果保持一致，在编译期间，不会对存在数据依赖关系的操作进行重排序



## java基础

- 接口是可以多继承的

- 对象锁和类锁：

  - 对象锁锁的是这个实例

  - 类锁锁的是这个类的class对象

    

- 使用线程

  - 实现runnable接口的run，在创建thread的时候，传入runnable对象

    ```java
    class MyRunnable implements Runnable{
      @Overide
      public void run(){
        //...
      }
    }
    public class MyMain{
      public static void main(String []args){
        MyRunnable instance = new MyRunnable();
        Thread thread = new Thread(instance);
        thread.start();
      }
    }
    ```

    

  - 实现callable接口，这个接口有返回类型，如：

    ```java
    public class MyCallable implements Callable<Integer> {
        public Integer call() {
            return 123;
        }
    }
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyCallable mc = new MyCallable();
        FutureTask<Integer> ft = new FutureTask<>(mc);
        Thread thread = new Thread(ft);
        thread.start();
        System.out.println(ft.get());
    }
    ```

    这个方法中，futureTask接收mc实现返回中间值操作，然后thread通过传入ft作为对象，创建thread类

    

  - 继承thread类，实现run方法（thread也即成了runnable）

    ```java
    public class MyThread extends Thread {
        public void run() {
            // ...
        }
    }
    ```

  - **不建议继承thread，建议实现接口**

    - Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
    - 类可能只要求可执行就行，继承整个 Thread 类开销过大？

- 在Java中，为了保证原子性，提供了两个高级的字节码指令`monitorenter`和`monitorexit`，这个是与监视器monitor有关，monitor不仅仅是充当一个互斥量的作用，还维护了安全性，在抛出异常的时候，monitor会将这个线程的锁进行释放

- 两种最常用的锁：reentry和sync

  - 优点
    - `ReentrantLock`可响应中断，内部用的是cas，`synchronized`是不可以响应中断的，为处理锁的不可用性提供了更高的灵活性，有一个锁升级的过程，最终是通过monitor维护一个互斥量来进行锁同步操作。
    - `ReentrantLock`可以实现公平锁
    - `ReentrantLock`可以配合条件变量进行有条件的唤醒
  - 缺点
    - 你没有释放锁，很难追踪到最初发生错误的位置，因为没有记录应该释放锁的位置和时间。网上找了一下，没有找到其他比较合理的答案，先暂且记住吧



## 实操

- 三个线程按顺序打印10次ABC

```java
package com.example.mafkademo.demo;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class CurrentTest {

    private static ReentrantLock lock = new ReentrantLock();
    private static Condition conditionA = lock.newCondition();
    private static Condition conditionB = lock.newCondition();
    private static Condition conditionC = lock.newCondition();
    private static volatile int state = 1;

    public static void main(String[] args) throws InterruptedException {

        ExecutorService poolService = Executors.newFixedThreadPool(3);
        Integer count = 10;
        poolService.execute(new Worker("A", count, 1, lock, conditionA, conditionB));
        poolService.execute(new Worker("B", count, 2, lock, conditionB, conditionC));
        poolService.execute(new Worker("C", count, 3, lock, conditionC, conditionA));
        Thread.sleep(5000);
        poolService.shutdownNow();
    }

    public static class Worker implements Runnable {
        private String key;
        private Integer count;
        private Lock lock;
        private Condition current;
        private Condition next;
        private int targetState;

        public Worker(String key, Integer count, int targetState, Lock lock, Condition cur, Condition next) {
            this.key = key;
            this.count = count;
            this.lock = lock;
            this.current = cur;
            this.next = next;
            this.targetState = targetState;
        }

        public void run() {
            this.lock.lock();
            try {
                for (int i = 0; i < count; i++) {
                    while (state != targetState) {
                        current.await();
                    }
                    System.out.println(i + "," + key);
                    state++;
                    if (state > 3) {
                        state = 1;
                    }
                    next.signal();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                this.lock.unlock();
            }
        }
    }
}
```



## volatile 关键字

- 不能解决原子性

- 解决的问题：

  - 有序性：禁止指令重排序,

    - `Load Barrier` 和 `Store Barrier`
    - 对于Load Barrier来说，在指令前插入Load Barrier，可以让高速缓存中的数据失效，强制从新从主内存加载数据
    - 对于Store Barrier来说，在指令后插入Store Barrier，能让写入缓存中的最新数据更新写入主内存，让其他线程可见。

  - 可见性：对于`volatile`变量，当对`volatile`变量进行写操作的时候，JVM会向处理器发送一条lock前缀的指令，将这个缓存中的变量回写到系统主存中。

    

- 缓存一致性协议

  - 每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成**无效状态**，当处理器要对这个数据进行修改操作的时候，会强制重新从系统内存里把数据读到处理器缓存里。
  - 缓存一致性让currenthasmap在读数据的时候不需要加锁也不会读取到脏数据。

- 内存可见性

  - 可见性是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

- 线程之间变量不可见原因：

  - Java内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，线程的工作内存中保存了该线程中是用到的**变量的主内存副本拷贝**，线程对变量的所有操作都必须在**工作内存**中进行，而不能直接读写主内存。
  - **不同的线程之间也无法直接访问对方工作内存中的变量**，线程间变量的传递均需要自己的工作内存和主存之间进行数据同步进行。所以，就可能出现线程1改了某个变量的值，但是线程2不可见的情况。






## 各种锁的适应场景

- 悲观：适合数据更改密集形，防止cpu轮训次数过多

- 乐观：适合数据更改不密集，省去上下文切换开销

- 读者优先读写锁：读线程可以插队写线程

- 写者优先读写锁：写线程可以插队读线程

- 锁细化：细化锁的粒度，比如数据库中的行锁其实就是一种细化的过程，以及单例模式的dcl写法，还有currenthashmap也是，这种做法可以避免一些没必要的的过程

  

- 锁粗化：粗化锁的粒度，为毛？

  - 因为锁的维护需要开销，如果对一个对象反复加锁
  - 如果虚拟机探测到有这样一串零碎的操作都对同一个对象加锁，将会把加锁同步的范围扩展（粗化）到整个操作序列的外部（由多次加锁编程只加锁一次）。
  - 通常情况下，为了保证多线程间的有效并发，会要求每个线程持有锁的时间尽可能短，但是大某些情况下，一个程序对同一个锁不间断、高频地请求、同步与释放，会消耗掉一定的系统资源，因为锁的讲求、同步与释放本身会带来性能损耗，这样高频的锁请求就反而不利于系统性能的优化了，虽然单次同步操作的时间可能很短。
  - **锁粗化就是告诉我们任何事情都有个度，有些情况下我们反而希望把很多次锁的请求合并成一个请求，以降低短时间内大量锁请求、同步、释放带来的性能损耗。**

- 细化vs粗化

  - 细化意味着减小锁的覆盖范围，因此往往意味着更短的锁持有时间，缺点在于导致更频繁的锁请求和锁释放，比如在for循环里面加锁，这会导致频繁的开销。
  - 粗化意味着增加锁的覆盖范围，因此往往意味着更长的锁持有时间，缺点在于导致并发量的降低，阻塞时间的增加，比如懒汉加锁的单例模式，就会导致即便不需要创建对象时也加了锁。

  

## Reentrantlock 的根基：AQS

- AQS核心思想是，如果被请求的共享资源空闲，那么就将当前请求资源的线程设置为有效的工作线程，将共享资源设置为锁定状态；如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中。
- 小心优先级翻转
- 一切都基于CAS
- ![img](https://p1.meituan.net/travelcube/82077ccf14127a87b77cefd1ccf562d3253591.png)
- 基础概念
  - AQS数据结构：
  - 同步状态：
  - 等待队列
  - 解锁过程
  - 中断机制
- 原理
  - 

## 并发的场景

- CPU密集

  - 考虑单核的情况

    ![img](https://upload-images.jianshu.io/upload_images/19895418-8a4d3c815c2abdb1)

    - 发现即便在理想状态下，多线程也没有比单线程快，何况还会有io开销

  - 考虑多核的情况

    - ![img](https://upload-images.jianshu.io/upload_images/19895418-7370e52c09df4d86)

- 可以得出感性结论，在cpu密集形的请求，线程数=cpu核心数，可以达到最大使用率

- IO密集形

  - 线程会因为io开销导致很长时间不占用cpu

  - ![img](https://upload-images.jianshu.io/upload_images/19895418-c2955cec5fbacf00)

  - 可以看到，如果不把io的时间填补上，cpu其实会有很多情况下处于空闲状态

  - 因此，理论上，最佳线程数就是
    $$
    \frac{1}{CPU利用率} = \frac{IO耗时}{CPU耗时}
    $$

- 经验上来说，总线程个数会比理论最佳值多1个


- 正则表达式
  - ^, $: 以xx开头，以xx结尾
  - .：匹配任意一个字符
  - +：1到多
  - *：0次以上
  - ？：匹配一个字符0～1次
  - [A-Za-z0-9]：利用中括号限定匹配自负
  - {a}, {a, b}, {a,}：逗号前面的是至少，右边的是至多
  - (xxx): 将多个字符捆绑成一个
  - 常见考题
    - 匹配邮箱
      - ^[A-Za-z0-9]+\.\@[A-Za-z0-9]+\.[A-Za-z0-9]+$
      - 不知道为什么+限定符不能在zsh中使用
    - 匹配ip
      - `^[1-2]{0,1}[0-9]{1,2}\.[1-2]{0,1}[0-9]{1,2}\.`

  





