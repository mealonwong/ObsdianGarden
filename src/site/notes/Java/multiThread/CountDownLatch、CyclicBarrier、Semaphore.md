---
{"dg-publish":true,"permalink":"/java/multi-thread/count-down-latch-cyclic-barrier-semaphore/"}
---


## CountDownLatch

> CountDownLatch是一个辅助器，允许一个或多个线程一直等待，直到一组线程全部准备就绪后同步执行，并互相等待其他线程的操作执行完成。

```Java
public CountDownLatch(int count) {  
    if (count < 0) throw new IllegalArgumentException("count < 0");  
    this.sync = new Sync(count);  
}
```
构造方法要求需要传入一个count值，用来计数，表示有多少组线程会同时执行。

CountDownLatch有两个常用的方法：

```Java
public void await() throws InterruptedException {  
    sync.acquireSharedInterruptibly(1);  
}

public void countDown() {  
    sync.releaseShared(1);  
}
```

当一个线程调用await方法时，该线程会被阻塞。每当有线程调用一次countDown方法，计数器就会被减一，当count的值被减为0的时候，被阻塞的线程才会继续执行。

## 源码

```Java
private static final class Sync extends AbstractQueuedSynchronizer {  
    private static final long serialVersionUID = 4982264981922014374L;  
  
    Sync(int count) {  
        setState(count);  
    }  
  
    int getCount() {  
        return getState();  
    }  
  
    protected int tryAcquireShared(int acquires) {  
        return (getState() == 0) ? 1 : -1;  
    }  
  
    protected boolean tryReleaseShared(int releases) {  
        // Decrement count; signal when transition to zero  
        for (;;) {  
            int c = getState();  
            if (c == 0)  
                return false;  
            int nextc = c - 1;  
            if (compareAndSetState(c, nextc))  
                return nextc == 0;  
        }  
    }  
}

```

### 实践

```Java
public class CountDownTest {  
    static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
    public static void main(String[] args) throws InterruptedException {  
        CountDownLatch latch = new CountDownLatch(2);  
        Worker w1 = new Worker("张三", 2000, latch);  
        Worker w2 = new Worker("李四", 3000, latch);  
        w1.start();  
        w2.start();  
  
        long startTime = System.currentTimeMillis();  
        latch.await();  
        System.out.println("bug全部解决，领导可以给客户交差了，任务总耗时："+ (System.currentTimeMillis() - startTime));  
  
    }  
  
    static class Worker extends Thread{  
        String name;  
        int workTime;  
        CountDownLatch latch;  
  
        public Worker(String name, int workTime, CountDownLatch latch) {  
            this.name = name;  
            this.workTime = workTime;  
            this.latch = latch;  
        }  
  
        @Override  
        public void run() {  
            System.out.println(name+"开始修复bug，当前时间："+sdf.format(new Date()));  
            doWork();  
            System.out.println(name+"结束修复bug，当前时间："+sdf.format(new Date()));  
            latch.countDown();  
        }  
  
        private void doWork() {  
            try {  
                //模拟工作耗时  
                Thread.sleep(workTime);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
}
```

## CyclicBarrier

> CyclicBarrier循环栅栏，一组线程会互相等待，直到所有线程都到达一个同步点。这个就非常有意思了，就像一群人被困到了一个栅栏前面，只有等最后一个人到达之后，他们才可以合力把栅栏（屏障）突破。

构造方法

```Java
public CyclicBarrier(int parties) {  
    this(parties, null);  
}

public CyclicBarrier(int parties, Runnable barrierAction) {  
    if (parties <= 0) throw new IllegalArgumentException();  
    this.parties = parties;  
    this.count = parties;  
    this.barrierCommand = barrierAction;  
}
```

第一个构造的参数，指的是需要几个线程一起到达，才可以使所有线程取消等待。第二个构造，额外指定了一个参数，用于在所有线程达到屏障时，优先执行 barrierAction。

## 实践
```Java
class BarrierTest {  
    public static void main(String[] args) {  
        CyclicBarrier barrier = new CyclicBarrier(2,() -> {  
            try {  
                System.out.println("等待裁判吹口哨");  
                Thread.sleep(1000);  
                System.out.println("裁判吹口哨");  
            }catch (Exception e){  
            }  
  
        });  //①  
        Runner runner1 = new Runner(barrier, "张三");  
        Runner runner2 = new Runner(barrier, "李四");  
        Runner runner3 = new Runner(barrier, "王五");  
        Runner runner4 = new Runner(barrier, "孙六");  
  
        ExecutorService service = Executors.newFixedThreadPool(2);  
        service.execute(runner1);  
        service.execute(runner2);  
        service.execute(runner3);  
        service.execute(runner4);  
  
        service.shutdown();  
  
    }  
  
}  
  
  
class Runner implements Runnable{  
  
    private CyclicBarrier barrier;  
    private String name;  
  
    public Runner(CyclicBarrier barrier, String name) {  
        this.barrier = barrier;  
        this.name = name;  
    }  
  
    @Override  
    public void run() {  
        try {  
            //模拟准备耗时  
            Thread.sleep(new Random().nextInt(5000));  
            System.out.println(name + ":准备OK");  
            barrier.await();  
            System.out.println(name +": 开跑");  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        } catch (BrokenBarrierException e){  
            e.printStackTrace();  
        }  
    }  
}
```

输出结果
```txt
李四:准备OK
张三:准备OK
等待裁判吹口哨
裁判吹口哨
张三: 开跑
李四: 开跑
王五:准备OK
孙六:准备OK
等待裁判吹口哨
裁判吹口哨
孙六: 开跑
王五: 开跑
```


## Semaphore

> Semaphore 信号量，用来控制同一时间，资源可被访问的线程数量，一般可用于流量的控制。

打个比方，现在有一段公路交通比较拥堵，那怎么办呢。此时，就需要警察叔叔出面，限制车的流量。

比如，现在有 20 辆车要通过这个地段， 警察叔叔规定同一时间，最多只能通过 5 辆车，其他车辆只能等待。只有拿到许可的车辆可通过，等车辆通过之后，再归还许可，然后把它发给等待的车辆，获得许可的车辆再通行，依次类推。

```Java
class SemaphoreTest {  
    private static int count = 20;  
  
    public static void main(String[] args) {  
  
        ExecutorService executorService = Executors.newFixedThreadPool(count);  
  
        //指定最多只能有五个线程同时执行  
        Semaphore semaphore = new Semaphore(5);  
  
        Random random = new Random();  
        for (int i = 0; i < count; i++) {  
            final int no = i;  
            executorService.execute(new Runnable() {  
                @Override  
                public void run() {  
                    try {  
                        //获得许可  
                        semaphore.acquire();  
                        System.out.println(no +":号车可通行");  
                        //模拟车辆通行耗时  
                        Thread.sleep(random.nextInt(2000));  
                        //释放许可  
                        semaphore.release();  
                    } catch (InterruptedException e) {  
                        e.printStackTrace();  
                    }  
                }  
            });  
        }  
  
        executorService.shutdown();  
  
    }  
}
```

这样每次就仅允许5辆车同时通行，但是此时存在一个问题就是只有5个通行证，那这5个通行证需要发给谁呢？
所以此时就需要用到锁了，既然大家都想要拿到通行证，那么就所有人同时去抢，谁能拿到就谁先走。
Semaphore提供了两个构造方法，其中有一个boolean的参数，可以控制锁是否公平，具体原理基于AQS实现。
```Java
/**默认采用非公平锁**/
public Semaphore(int permits) {  
    sync = new NonfairSync(permits);  
}

public Semaphore(int permits, boolean fair) {  
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);  
}
```

公平锁就是按照先来后到的顺序，谁先来的谁先走，非公平锁就是有一个后来的车，但是是个救护车拥有优先通过权，所以不需要排队可以直接通过。