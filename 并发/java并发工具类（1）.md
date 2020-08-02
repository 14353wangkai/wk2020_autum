## CountDownLatch

![img](https://camo.githubusercontent.com/c2a94b75d7379c204996f24411a9c497125cfa06/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f62613037383239312d373931652d343337382d623664312d6563653736633266306231342e706e67)

- Countdownlatch: 为每一个调用了await的线程提供一个门闩，只有当countDown数值降低到0的时候，被await阻塞的线程才能被唤醒
- 多线程计数的时候记住给主线程加上门闩，否则可能遇到没有计数完成就调用主线程的print方法的情况。
- countdown计数只会计算一次，这个方法是cas实现的，调用了aqs中的doReleaseShared方法，这个方法是通过cas将当前数值-1的



## CyclicBarrier

- 用于强制要求n个线程都到达的时候，这些线程才可以运行。就像体育课报数一样，一个班级48个人，当报数从48到1一个没拉下的时候才开始上课
- cyclicbarrier提供了reset方法，可以让计数器恢复
- CyclicBarrier.await()用于让计数器-1

![img](https://camo.githubusercontent.com/10dd07a9c7828fab8f68a0f953755869dc286a8e/68747470733a2f2f63732d6e6f7465732d313235363130393739362e636f732e61702d6775616e677a686f752e6d7971636c6f75642e636f6d2f66373161663636622d306435342d343339392d613434622d6634376235383332313938342e706e67)

## Semaphore

- 用于限制能够同时获取资源的线程数量，是互斥锁的升级版本
- semaphore通过静态函数, acquire和release来设置信号量的大小，当acquire成功的时候，信号量-1, 当release的时候，信号量+1，这两个操作必然是原子的



## BlockingQueue

- 阻塞队列，常用于生产者消费者模型，这机制本身就是一种消息队列
- 提供了阻塞的 take() 和 put() 方法：如果队列为**空 take() 将阻塞**，直到队列中有内容；如果**队列为满 put() 将阻塞**，直到队列有空闲位置。

- 用阻塞队列来实现生产者消费者模型，和用linkedhashmap实现lru是一个性质，不需要自己再加额外判断



## FutureTask

- 实现了future和runnable接口

  - 其中runnable是用于运行程序，future负责保存运行过程中线程保留的数据

  - future：判断任务是否完成，中断任务，获取执行结果，futuretask是future的唯一一个实现类

    ```java
    public interface Future<V> {
        boolean cancel(boolean mayInterruptIfRunning);
        boolean isCancelled();
        boolean isDone();
        V get() throws InterruptedException, ExecutionException;
        V get(long timeout, TimeUnit unit)
            throws InterruptedException, ExecutionException, TimeoutException;
    }
    ```

    - Cancel：取消任务，返回值代表取消成功与否，如果线程已经完成，则返回false，如果线程还没开始，返回true
      - 参数：用于判断程序运行时能否被打断。如果传入true，则在运行中线程是可以中断的，传入false，则不可以。
    - isCancelled方法表示任务是否被取消成功
    - isDone方法表示任务是否已经完成 
    - get()方法用来获取执行结果，这个方法会**阻塞**，会一直等到任务执行完毕才返回；
    - get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

  

- 例子

```java
public class FutureTaskExample {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int result = 0;
                for (int i = 0; i < 100; i++) {
                    Thread.sleep(10);
                    result += i;
                }
                return result;	//futuretask封装的结果
            }
        });

        Thread computeThread = new Thread(futureTask);
        computeThread.start();

        Thread otherThread = new Thread(() -> {
            System.out.println("other task is running...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        otherThread.start();
        System.out.println(futureTask.get());
    }
}
```



