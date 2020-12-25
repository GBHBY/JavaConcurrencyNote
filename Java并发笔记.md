# 并发

## 简介

- Java的线程机制是抢占式的，也就是说，调度机制会周期性的中断某个线程，切换到其他的线程。

## 定义任务

- 最基础的创建一个线程（也就是定义一个任务）通过继承Thread

- 也可以实现Runnable接口

  - ````java 
    public class LiftOff implements Runnable {
      protected int countDown = 10; // Default
      private static int taskCount = 0;
      private final int id = taskCount++;
      public LiftOff() {}
      public LiftOff(int countDown) {
        this.countDown = countDown;
      }
      public String status() {
        return "#" + id + "(" +
          (countDown > 0 ? countDown : "Liftoff!") + "), ";
      }
      public void run() {
        while(countDown-- > 0) {
          System.out.print(status());
          Thread.yield();
        }
      }
    } ///:~
    ````

    - `Thread.yield()`是用来建议`线程调度机制：建议你去执行其他的线程`，但这仅仅是*建议*，线程调度机制并不一定会去执行其他线程
    
  - 以上两个都不是建议使用的创建线程的办法，以后会描述如何正确的创建线程

  

## Thread

- 上文的代码并不是开始一个线程，仅仅是展示了这个线程如果执行的话，会进行的操作，而下面的代码是如何启动一个任务也就是线程

  - ````java
    public class BasicThreads {
      public static void main(String[] args) {
        Thread t = new Thread(new LiftOff());
        t.start();
        System.out.println("Waiting for LiftOff");
      }
    } /* Output: (90% match)
    Waiting for LiftOff
    #0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(Liftoff!),
    *///:~
    
    ````

  - 第4行是启动了一个t线程，但是这个程序中一共有两个线程，一个是t，另一个是main线程，这两个线程是同时进行的（实际上并不是同时执行的，在单位时间内，仅有一个线程执行，但因为cpu执行速度很快，我们看起来是同时执行的）

## Executor

- juc包中的Executor会帮助我们管理Thread对象，简化并发编程

- 通过`newCachedThreadPool()`来创建线程

  - ````java 
    import java.util.concurrent.*;
    
    public class CachedThreadPool {
        public static void main(String[] args) {
            ExecutorService exec = Executors.newCachedThreadPool();
            for (int i = 0; i < 5; i++) {
                exec.execute(new LiftOff());
            }
            exec.shutdown();
        }
    } /* Output: (Sample)
    #0(9), #0(8), #1(9), #2(9), #3(9), #4(9), #0(7), #1(8), #2(8), #3(8), #4(8), #0(6), #1(7), #2(7), #3(7), #4(7), #0(5), #1(6), #2(6), #3(6), #4(6), #0(4), #1(5), #2(5), #3(5), #4(5), #0(3), #1(4), #2(4), #3(4), #4(4), #0(2), #1(3), #2(3), #3(3), #4(3), #0(1), #1(2), #2(2), #3(2), #4(2), #0(Liftoff!), #1(1), #2(1), #3(1), #4(1), #1(Liftoff!), #2(Liftoff!), #3(Liftoff!), #4(Liftoff!),
    *///:~
    ````

  - 当然，上面这个也不是最正确的定义线程的方式

  - `exec.shutdown()`这个方法是用来停止该线程的，而` exec.execute()`，这个方法的参数是实现`Runnable`接口的类是用来产生线程的方法，本例就是产生5个线程，并执行LiftOff中run方法。

- 通过`newFixedThreadPool`来创建线程，这个可以指定创建线程的数量。这么做可以节省时间，因为不用为每个线程都固定的付出创建线程的开销

  - ````java 
    public class FixedThreadPool {
      public static void main(String[] args) {
        // Constructor argument is number of threads:
        ExecutorService exec = Executors.newFixedThreadPool(5);
        for(int i = 0; i < 5; i++)
          exec.execute(new LiftOff());
        exec.shutdown();
      }
    } /* Output: (Sample)
    #0(9), #0(8), #1(9), #2(9), #3(9), #4(9), #0(7), #1(8), #2(8), #3(8), #4(8), #0(6), #1(7), #2(7), #3(7), #4(7), #0(5), #1(6), #2(6), #3(6), #4(6), #0(4), #1(5), #2(5), #3(5), #4(5), #0(3), #1(4), #2(4), #3(4), #4(4), #0(2), #1(3), #2(3), #3(3), #4(3), #0(1), #1(2), #2(2), #3(2), #4(2), #0(Liftoff!), #1(1), #2(1), #3(1), #4(1), #1(Liftoff!), #2(Liftoff!), #3(Liftoff!), #4(Liftoff!),
    *///:~
    ````

- `SingelThreadExecutor`可以理解为参数为1的`FixedThreadPool`。这个创建线程的方法适用于那些长时间存活的任务。

  - ````java 
    import java.util.concurrent.*;
    
    public class SingleThreadExecutor {
      public static void main(String[] args) {
        ExecutorService exec =
          Executors.newSingleThreadExecutor();
        for(int i = 0; i < 5; i++)
          exec.execute(new LiftOff());
        exec.shutdown();
      }
    } /* Output:
    #0(9), #0(8), #0(7), #0(6), #0(5), #0(4), #0(3), #0(2), #0(1), #0(Liftoff!), #1(9), #1(8), #1(7), #1(6), #1(5), #1(4), #1(3), #1(2), #1(1), #1(Liftoff!), #2(9), #2(8), #2(7), #2(6), #2(5), #2(4), #2(3), #2(2), #2(1), #2(Liftoff!), #3(9), #3(8), #3(7), #3(6), #3(5), #3(4), #3(3), #3(2), #3(1), #3(Liftoff!), #4(9), #4(8), #4(7), #4(6), #4(5), #4(4), #4(3), #4(2), #4(1), #4(Liftoff!),
    *///:~
    ````

  - 本例中提交了多个由`SingleThreadExecutor`创建的线程，这些线程是一个一个创建的，而不是像`FixedThreadPool`那样一次性创建。

  - 假设有大量的线程，这些线程的任务就是使用文件系统。那么就可以使用`SingleThreadExecutor`来创建线程，因为这样可以确保任意时刻在任何线程中只有一个任务在使用文件系统，这样就不需要来对文件系统进行同步。当然更好的方式就是同步资源。

  

## Callable

- `Runnable`只能是执行任务，但是不能产生返回值。如果希望任务结束时，产生返回值，那么就要实现`Callable`,

  - ````java 
    import java.util.concurrent.*;
    import java.util.*;
    
    class TaskWithResult implements Callable<String> {
      private int id;
      public TaskWithResult(int id) {
        this.id = id;
      }
      public String call() {
        return "result of TaskWithResult " + id;
      }
    }
    
    public class CallableDemo {
      public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        ArrayList<Future<String>> results =
          new ArrayList<Future<String>>();
        for(int i = 0; i < 10; i++)
          results.add(exec.submit(new TaskWithResult(i)));
        for(Future<String> fs : results)
          try {
            // get() blocks until completion:
            System.out.println(fs.get());
          } catch(InterruptedException e) {
            System.out.println(e);
            return;
          } catch(ExecutionException e) {
            System.out.println(e);
          } finally {
            exec.shutdown();
          }
      }
    } /* Output:
    result of TaskWithResult 0
    result of TaskWithResult 1
    result of TaskWithResult 2
    result of TaskWithResult 3
    result of TaskWithResult 4
    result of TaskWithResult 5
    result of TaskWithResult 6
    result of TaskWithResult 7
    result of TaskWithResult 8
    result of TaskWithResult 9
    *///:~
    ````

## 休眠

- ````java 
  import java.util.concurrent.*;
  
  public class SleepingTask extends LiftOff {
    public void run() {
      try {
        while(countDown-- > 0) {
          System.out.print(status());
          // Old-style:
          // Thread.sleep(100);
          // Java SE5/6-style:
          TimeUnit.MILLISECONDS.sleep(100);
        }
      } catch(InterruptedException e) {
        System.err.println("Interrupted");
      }
    }
    public static void main(String[] args) {
      ExecutorService exec = Executors.newCachedThreadPool();
      for(int i = 0; i < 5; i++)
        exec.execute(new SleepingTask());
      exec.shutdown();
    }
  } /* Output:
  #0(9), #1(9), #2(9), #3(9), #4(9), #0(8), #1(8), #2(8), #3(8), #4(8), #0(7), #1(7), #2(7), #3(7), #4(7), #0(6), #1(6), #2(6), #3(6), #4(6), #0(5), #1(5), #2(5), #3(5), #4(5), #0(4), #1(4), #2(4), #3(4), #4(4), #0(3), #1(3), #2(3), #3(3), #4(3), #0(2), #1(2), #2(2), #3(2), #4(2), #0(1), #1(1), #2(1), #3(1), #4(1), #0(Liftoff!), #1(Liftoff!), #2(Liftoff!), #3(Liftoff!), #4(Liftoff!),
  *///:~
  
  ````

- 上面的任务是完美的按照0->4然后4->0.因为每个线程执行打印之后都会睡眠也就是阻塞，这会使线程调度机制去执行其他的线程。

- ，上面的例子是把Runnable例子中的`Thread.yield()`换成了`TimeUnit.MILLISECONDS.sleep(100);`,也就是休眠。可以看出区别。并且，休眠指令是一定会执行的，但是yield不一定。

## 优先级

- 线程的优先级将该线程的重要性传递给了调度器。尽管CPU处理现有线程集的顺序不是确定的，但是调度器将倾向于让优先级高的线程先执行。但不意味着优先级低的线程不会执行，，优先级低的线程只是执行的频率比较低。

- 尽可能的不要改变线程的优先级

- 示例

  - ````java 
    import java.util.concurrent.*;
    
    public class SimplePriorities implements Runnable {
      private int countDown = 5;
      private volatile double d; // No optimization
      private int priority;
      public SimplePriorities(int priority) {
        this.priority = priority;
      }
      public String toString() {
        return Thread.currentThread() + ": " + countDown;
      }
      public void run() {
        Thread.currentThread().setPriority(priority);
        while(true) {
          // An expensive, interruptable operation:
          for(int i = 1; i < 100000; i++) {
            d += (Math.PI + Math.E) / (double)i;
            if(i % 1000 == 0)
              Thread.yield();
          }
          System.out.println(this);
          if(--countDown == 0) return;
        }
      }
      public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        for(int i = 0; i < 5; i++)
          exec.execute(
            new SimplePriorities(Thread.MIN_PRIORITY));
        exec.execute(
            new SimplePriorities(Thread.MAX_PRIORITY));
        exec.shutdown();
      }
    } /* Output: (70% match)
    Thread[pool-1-thread-6,10,main]: 5
    Thread[pool-1-thread-6,10,main]: 4
    Thread[pool-1-thread-6,10,main]: 3
    Thread[pool-1-thread-6,10,main]: 2
    Thread[pool-1-thread-6,10,main]: 1
    Thread[pool-1-thread-3,1,main]: 5
    Thread[pool-1-thread-2,1,main]: 5
    Thread[pool-1-thread-1,1,main]: 5
    Thread[pool-1-thread-5,1,main]: 5
    Thread[pool-1-thread-4,1,main]: 5
    ...
    *///:~
    ````

  - JDK有10个优先级，但是与每个操作系统的映射是不固定的，所以当调整优先级的时候，只是用**MAX_PRIORITY**,**NORM_PRIORITY**,**MIN_PRIORITY**。

  

  









