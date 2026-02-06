# 并发编程知识汇总

## 1. 多线程基础

### 1.1 创建线程的三种方式

1. **继承 Thread 类**：创建一个类继承 `Thread` 类，并重写 `run` 方法。
   ```java
   public class MyThread extends Thread {
       @Override
       public void run() {
           System.out.println(getName() + ": 执行任务");
       }
   }
   ```
2. **实现 Runnable 接口**：创建一个类实现 `Runnable` 接口，并重写 `run` 方法。更灵活，易于与线程池配合。
   ```java
   public class MyRunnable implements Runnable {
       @Override
       public void run() {
           System.out.println(Thread.currentThread().getName() + ": 执行任务");
       }
   }
   ```
3. **实现 Callable 接口 (FutureTask)**：实现 `Callable` 接口，重写 `call` 方法。这种方式可以通过 `FutureTask` 获取任务执行的返回值。
   ```java
   public class CallerTask implements Callable<String> {
       public String call() throws Exception {
           return "Hello, task result!";
       }
   }
   // 使用方式
   FutureTask<String> task = new FutureTask<>(new CallerTask());
   new Thread(task).start();
   String result = task.get(); // 获取结果
   ```

| 创建方式                   | 优点                                                 | 缺点                                                            |
| :------------------------- | :--------------------------------------------------- | :-------------------------------------------------------------- |
| **实现 Runnable/Callable** | 避免 Java 单继承局限；适合资源共享；解耦线程与任务。 | 编程稍微复杂，如需访问当前线程需使用 `Thread.currentThread()`。 |
| **继承 Thread**            | 编程简单，直接使用 `this` 即可获取当前线程。         | 受限于 Java 单继承；不适合多线程处理同一资源。                  |

### 1.2 run() 与 start() 的区别

* **为什么要重写 run()**：默认的 `run()` 方法不执行任何操作。为了让线程执行具体任务，必须重写它。
* `run()`：封装线程执行的代码，直接调用相当于在主线程中执行普通方法。
* `start()`：启动线程，使线程进入“就绪”状态，由 JVM 在合适时机调用此线程的 `run()` 方法。**只有 start() 才能真正开启多线程。**

## 2. 获取线程执行结果 (Future)

### 2.1 Future 接口

提供了异步计算的各种功能：

* `cancel(boolean mayInterrupt)`：尝试取消任务。若任务已完成或无法取消则返回 false。
* `isCancelled()`：任务是否在正常完成前被取消。
* `isDone()`：任务是否已完成（包括正常完成、异常、取消）。
* `get()`：**阻塞**获取结果，直到任务完成。
* `get(timeout, unit)`：在指定时间内获取结果，超时则返回 null 或抛异常。

### 2.2 FutureTask

`FutureTask` 实现了 `RunnableFuture` 接口（继承自 `Runnable` 和 `Future`），既可以作为 `Runnable` 被线程执行，又可以作为 `Future` 获取结果。

## 3. 线程运行原理

### 2.1 栈与栈帧

* 线程启动后，虚拟机会为每个线程分配一段**栈内存**。
* 栈由多个**栈帧**（Stack Frame）构成，对应着每次方法调用时占用的内存。
* 每个线程只能有一个**活动栈帧**，对应着当前正在执行的那个方法。

### 2.2 线程上下文切换（Context Switch）

发生情况：

* 线程的 CPU 时间片用完。
* 垃圾回收。
* 有更高优先级的线程需要运行。
* 线程自己调用了 `sleep`、`yield`、`wait`、`join`、`park`、`synchronized`、`lock` 等方法。

当 Context Switch 发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java 中对应的概念就是**程序计数器**（Program Counter Register），它的作用是记住下一条 jvm 指令的执行地址，是线程私有的。

## 3. 查看进程与线程

### 3.1 操作系统命令

* **Windows**:
  * `tasklist`: 查看所有进程。
  * `taskkill /F /PID <pid>`: 强制杀死进程。
* **Linux**:
  * `ps -fe`: 查看所有进程。
  * `top`: 动态查看进程信息。
  * `kill <pid>`: 杀死进程。

### 3.2 Java 工具

* `jps`: 查看所有 Java 进程。
* `jstack <PID>`: 查看某个 Java 进程内的线程堆栈信息（排查死锁、CPU 飙高常用）。
* `jconsole`: 图形化界面，查看 Java 进程内线程的运行情况。

## 4. 线程常用方法

| 方法名                            | 作用                 | 说明                                                                                   |
| :-------------------------------- | :------------------- | :------------------------------------------------------------------------------------- |
| `start()`                         | 启动新线程           | 每个线程只能调用一次（**不可重复调用**）。                                             |
| `run()`                           | 线程启动后执行的逻辑 | 直接调用不会启动新线程。                                                               |
| `join()` / `join(n)`              | 等待线程结束         | 使调用方等待该线程结束（或超时）。                                                     |
| `sleep(n)`                        | 线程休眠             | 状态变为 `TIMED_WAITING`，**不释放锁**。建议在 `while(true)` 循环中调用以防 CPU 空转。 |
| `yield()`                         | 提示让出 CPU         | 状态变为 `RUNNABLE`，具体是否让出依赖调度器。                                          |
| `interrupt()`                     | 打断线程             | 设置打断标记；若线程在 sleep/wait/join 会抛异常并清除标记。                            |
| `isInterrupted()`                 | 判断是否被打断       | 不会清除打断标记。                                                                     |
| `Thread.interrupted()`            | 判断是否被打断       | **静态方法，会清除打断标记**。                                                         |
| `setDaemon(true)`                 | 设置为守护线程       | 主线程结束，守护线程也会被强制结束（如 GC 线程）。                                     |
| `getId()`                         | 获取线程唯一 ID      | 线程的长整型标识符。                                                                   |
| `getName()` / `setName()`         | 获取/修改线程名      | 方便调试和排查问题。                                                                   |
| `getPriority()` / `setPriority()` | 获取/修改优先级      | 范围 1-10，具体实现依赖任务调度器。                                                    |
| `isAlive()`                       | 线程是否存活         | 线程已启动且尚未结束。                                                                 |
| `currentThread()`                 | 获取当前线程         | 静态方法。                                                                             |

### 4.1 过时方法（不推荐使用）

以下方法由于容易导致死锁或资源无法释放，已不推荐使用：

* `stop()`: 强制停止线程，可能导致同步资源未释放。
* `suspend()` / `resume()`: 挂起和恢复线程，极易导致死锁。

### 4.2 两阶段终止模式

如何优雅地在一个线程中终止另一个线程？利用 `interrupt`。
![1764300709161](image/1764300709161.png)

## 5. 线程生命周期与状态

![1762734021190](image/1762734021190.png)

### 5.1 操作系统层面（五种状态）

![1764301356509](image/1764301356509.png)

1. **初始状态**: 已创建但未与操作系统关联。
2. **可运行状态**: 等待 CPU 调度。
3. **运行状态**: 正在执行，时间片用完会回到可运行状态。
4. **阻塞状态**: 调用阻塞 API（如读写文件），结束后回到可运行状态。
5. **终止状态**: 运行完毕。

### 5.2 Java 层面（Thread.State 六种状态）

![1762959066356](image/1762959066356.png)

```java
public enum State {
    NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED;
}
```

![1764301549678](image/1764301549678.png)

| **线程状态**               | **导致状态发生条件**                                                                                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| NEW（新建）                | 线程刚被创建，但是并未启动，还没调用 start 方法，只有线程对象，没有线程特征                                                                                        |
| Runnable（可运行）         | 线程可以在 Java 虚拟机中运行的状态，可能正在运行自己代码，也可能没有<br />这取决于操作系统处理器，调用了 t.start() 方法                                            |
| Blocked（阻塞）            | 当一个线程试图获取一个对象锁，而该对象锁被其他的线程持有，则该线程进入 Blocked 状态<br />当该线程持有锁时，该线程将变成 Runnable 状态                              |
| Waiting（无限等待）        | 一个线程在等待另一个线程执行一个（唤醒）动作时，该线程进入 Waiting 状态<br />进入这个状态后不能自动唤醒，必须等待另一个线程调用 notify 或者 notifyAll 方法才能唤醒 |
| Timed Waiting （限期等待） | 有几个方法有超时参数，调用将进入 Timed Waiting 状态<br />这一状态将一直保持到超时期满或者接收到唤醒通知。<br />带有超时参数的常用方法有 Thread.sleep 、Object.wait |
| Teminated（结束）          | run 方法正常退出而死亡，或者因为没有捕获的异常终止了 run 方法而死亡                                                                                                |

* **NEW → RUNNABLE**
  * 当调用 `t.start()` 方法时，由 **NEW → RUNNABLE**
* **RUNNABLE <--> WAITING**
  * 调用 `obj.wait()`方法时
  * 调用 `obj.notify()`、`obj.notifyAll()`、`t.interrupt()`：
    * 竞争锁成功，t 线程从 **WAITING → RUNNABLE**
    * 竞争锁失败，t 线程从 **WAITING → BLOCKED**
  * 当前线程调用 `t.join()` 方法，注意是当前线程在 t 线程对象的监视器上等待：**RUNNABLE → WAITING**
  * 当前线程调用 `LockSupport.park()` 方法：**RUNNABLE → WAITING，`unpark(线程)` 反之**
* **RUNNABLE <--> TIMED_WAITING**
  * 调用 `obj.wait(long n)` 方法、当前线程调用 `t.join(long n)` 方法、当前线程调用 `Thread.sleep(long n)` 方法、当前线程调用 `LockSupport.parkNanos(long nanos)` 方法
* **RUNNABLE <--> BLOCKED**
  * t 线程用 `synchronized(obj)` 获取了对象锁时竞争失败
* **RUNNABLE --> TERMINATED**
  * 线程所有代码执行完毕

### 5.3 Java 线程和 OS 线程状态比较

| Java 线程状态 | OS 线程状态   |
| ------------- | ------------- |
| NEW           | New           |
| RUNNABLE      | Ready/Running |
| BLOCKED       | Blocked       |
| WAITING       | Waiting       |
| TIMED_WAITING | Waiting       |
| TERMINATED    | Terminated    |

## 6. 共享资源问题

### 6.1 问题引入

```java
static int counter = 0;

public static void main(String[] args) throws InterruptedException{
    Thread t1 = new Thread(() ->{
        for(int i = 0;i < 1000;i++){
            counter++;
        }
    });

    Thread t2 = new Thread(() ->{
        for(int i = 0;i < 1000;i++){
            counter--;
        }
    });

    t1.start();
    t2.start();
    t1.join();
    t2.join();
    log.info("{}",counter);
}
```

* 上面代码计算结果可能是0，正数或负数
* 单线程没问题：

![1768890582392](image/并发编程核心知识汇总/1768890582392.png)

* 多线程可能的情况：

![1768890609490](image/并发编程核心知识汇总/1768890609490.png)

![1768890622051](image/并发编程核心知识汇总/1768890622051.png)

### 6.2 核心概念

**6.2.1 临界区**

**临界区**：一段代码内部存在对**共享资源**的**多线程读写操作**

**6.2.2 竞态条件**

**竞态条件**：多个线程在临界区执行，由于代码执行序列不同导致结果无法预测的情况

### 6.3 synchronized 解决方案

**6.3.1 基本特性**

- **阻塞式解决方案**：采用互斥方式让同一时刻最多有一个线程持有**对象锁**
- **互斥和同步**：都可以用 synchronized 解决
  - **互斥**：保证临界区竞态条件，同一时刻只能有一个线程执行临界区代码
  - **同步**：线程执行先后顺序不同，需要一个线程等待其他线程

**6.3.2 语法**

```java
synchronized(对象){
    临界区
}
```

**6.3.3 使用示例**

```java

  static final Object room = new Object();

  public static void main(String[] args) throws InterruptedException{
      Thread t1 = new Thread(() ->{
          for(int i = 0;i < 1000;i++){
              synchronized(room){
                  counter++;
              }
          }
      });

      Thread t2 = new Thread(() ->{
         for(int i = 0;i < 1000;i++){
              synchronized(room){
                  counter--;
              }
          }
      });

      t1.start();
      t2.start();
      t1.join();
      t2.join();
      log.info("{}",counter);
  }

  class Room{
      private int counter = 0;

      public void increment(){
           synchronized(room){
              counter++;
          }
      }  

      public void decrement(){
           synchronized(room){
              counter--;
          }
      }

      public int getCounter(){
           synchronized(room){
              return counter;
          }
      }
  }
```

  ![1769094927786](image/并发编程知识/1769094927786.png)

![1769094945703](image/并发编程知识/1769094945703.png)

### 6.4 线程安全分析

**6.4.1 成员变量与静态变量**

* **没有被共享**：线程安全
* **被共享**
  * 只有读操作：线程安全
  * 有读写操作：线程不安全
* **添加 final**：线程安全

**6.4.2 局部变量**

* **局部变量本身**：线程安全
* **局部变量引用的对象**：线程不一定安全
  * 对象没有逃离方法作用域：线程安全
  * 对象逃离方法作用域：不安全

```java
public static void test(){
    int i = 10;
    i++;
}
```

* 每个线程调用 test 方法时，会在线程栈帧中创建多份，变量 i 不存在共享，不存在线程安全问题

### 6.5 常见线程安全类

**6.5.1 线程安全的类**

* **String**：创建新字符串
* **基本数据包装对象类**：如 Integer、Long 等
* **StringBuffer**
* **Random**
* **java.util.concurrent 包下的类**

**6.5.2 组合操作的非原子性**

这些类的每个方法是原子的，但是它们多个方法的**组合不是原子的**

```java
HashTable table = new HashTable();
if(table.get("key")==null){
    table.put("key",value);
}
```

![1769097269261](image/并发编程知识/1769097269261.png)

### 6.6 synchronized 底层原理

**6.6.1 Java 对象头**

* **32 位虚拟机为例**
* **Klass Word**：表示对象的类型，指向对象从属的 class

![1769223450253](image/并发编程知识/1769223450253.png)

* **Mark Word**

![1769223556056](image/并发编程知识/1769223556056.png)

* **Normal 正常状态**：
  * hashcode：哈希码
  * age：垃圾回收时的分代年龄
  * biased_lock：偏向锁
  * 01：偏向锁状态
* **Biased ：**
* **Lightweight Locked 轻量级锁：**
  * ptr_to_lock_record：锁记录的地址
* **Heavyweight Locked 重量级锁：**
  * ptr_to_heavyweight_monitor：指向 monitor 指针
  * 10：偏向锁状态
* **Marked for GC 回收状态**

**6.6.2 Monitor（监视器/管程）**

* 每个 Java 对象都关联一个 Monitor 对象
* 使用 synchronized 给对象上锁，其对象头 Mark Word 指向 Monitor 对象指针
* 拿到锁的指向 Monitor 的 `Owner` 字段，没拿到锁的指向 `EntryList` 字段

### 6.7 synchronized 优化原理

**6.7.1 轻量级锁**

* 多线程访问时间错开了（没有竞争），可以用轻量级锁优化
* 轻量级锁语法仍是 synchronized
* 轻量锁发生竞争会升级为重量级锁
* 创建锁记录对象，每个线程栈帧都包含一个锁记录，存储锁定对象的 Mark Word

![1769228362066](image/并发编程知识/1769228362066.png)

* 锁记录 Object reference 指向锁对象，尝试用 cas 替换 Object 的 Mark Word，将其值写存入锁记录

![1769228432282](image/并发编程知识/1769228432282.png)

* 如果替换成功，对象头存储锁记录的地址和状态 00

![1769228459344](image/并发编程知识/1769228459344.png)

* cas 失败
  * 其他线程已经持有该锁，进入锁膨胀过程
  * 自己执行 synchronized 锁重入，再添加一个锁记录作为重入计数，交换必定失败
  * ![1769228628063](image/并发编程知识/1769228628063.png)
  * 退出 synchronized（解锁）时，如果有取值为 null 的锁记录，表示有重入，重置锁记录表示重入计数减一
  * ![1769228776365](image/并发编程知识/1769228776365.png)
  * 退出 synchronized（解锁）时，锁记录不为 null，使用 cas 将 Mark Word 的值恢复给对象头
  * ![1769228836622](image/并发编程知识/1769228836622.png)
  * 成功，则解锁成功；失败，则说明轻量级锁进行了锁膨胀，进入重量级锁解锁过程

**6.7.2 锁膨胀**

* 加轻量级锁时 CAS 失败，说明有锁竞争，进行锁膨胀，轻量级锁升级为重量级锁
* ![1769229013325](image/并发编程知识/1769229013325.png)
* Thread-1 加轻量级锁失败，进入锁膨胀过程
  * 为 Object 对象申请 Monitor 锁，Object 指向重量级锁地址
  * 进入 Monitor 的 EntryList，进入 BLOCKED 状态

![1769229074564](image/并发编程知识/1769229074564.png)

* Thread-0 解锁失败，进入重量级解锁流程，按照 Monitor 地址找到指向对象，设置 Owner 为 null，唤醒 EntryList 中 BLOCKED 状态的线程

**6.7.3 自旋优化**

* 先不进入阻塞状态，而是循环等待一会，如果此时锁被释放，则可以避免阻塞（阻塞需要一次上下文切换，消耗性能）
* 多核 CPU 才建议自旋，单核自旋纯粹浪费

### 6.8 wait-ify

**6.8.1 原理**

* Owner 线程发现条件不满足，调用 wait 方法，即可进入 WaitSet 变为 WAITING 状态
* BLOCKED 和 WAITING 的线程都处于阻塞状态，不占用 CPU 时间片
* BLOCKED 线程会在 Owner 线程释放锁时唤醒
* WAITING 线程会在 Owner 线程调用 notify 或 notifyAll 时唤醒，唤醒后并不意味者立刻获得锁，**需要进入 EntryList 重新竞争**

![1770201367426](image/并发编程知识/1770201367426.png)

**6.8.2 API**

* Object 类 API
* 必须要先获得对象所才能调用

```java
public final void notify():唤醒正在等待对象监视器的单个线程。
public final void notifyAll():唤醒正在等待对象监视器的所有线程。
public final void wait():导致当前线程等待，直到另一个线程调用该对象的 notify() 方法或 notifyAll()方法。
public final native void wait(long timeout):有时限的等待, 到n毫秒后结束等待，或是被唤醒
```

* **对比 sleep()：**
  * sleep 是 Thread 方法，wait 是 Object 方法
  * sleep 不需要强制和 synchronized 配合使用，但是 wait 需要
  * sleep 睡眠时不会释放对象锁，wait 等待时会释放

**6.8.3 代码示例**

* 虚假唤醒：notify 只能随机唤醒一个 WaitSet 中的线程，这时如果有其它线程也在等待，那么就可能唤醒不了正确的线程
  * 解决方法：采用 notifyAll
* notifyAll 仅解决某个线程的唤醒问题，使用 if + wait 判断仅有一次机会，一旦条件不成立，无法重新判断
  * 解决方法：用 while + wait，当条件不成立，再次 wait

```java
@Slf4j(topic = "c.demo")
public class demo {
    static final Object room = new Object();
    static boolean hasCigarette = false;    //有没有烟
    static boolean hasTakeout = false;

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            synchronized (room) {
                log.debug("有烟没？[{}]", hasCigarette);
                while (!hasCigarette) {//while防止虚假唤醒
                    log.debug("没烟，先歇会！");
                    try {
                        room.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                log.debug("有烟没？[{}]", hasCigarette);
                if (hasCigarette) {
                    log.debug("可以开始干活了");
                } else {
                    log.debug("没干成活...");
                }
            }
        }, "小南").start();

        new Thread(() -> {
            synchronized (room) {
                Thread thread = Thread.currentThread();
                log.debug("外卖送到没？[{}]", hasTakeout);
                if (!hasTakeout) {
                    log.debug("没外卖，先歇会！");
                    try {
                        room.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                log.debug("外卖送到没？[{}]", hasTakeout);
                if (hasTakeout) {
                    log.debug("可以开始干活了");
                } else {
                    log.debug("没干成活...");
                }
            }
        }, "小女").start();


        Thread.sleep(1000);
        new Thread(() -> {
        // 这里能不能加 synchronized (room)？
            synchronized (room) {
                hasTakeout = true;
				//log.debug("烟到了噢！");
                log.debug("外卖到了噢！");
                room.notifyAll();
            }
        }, "送外卖的").start();
    }
}
```

* 代码套路

```java
synchronized(lock){
    while(条件不成立){
        lock.wait();
    }
    // 执行业务
}

// 另一个线程
```

### 6.9 保护性暂停

**6.9.1 单任务版**

Guarded Suspension，用在一个线程等待另一个线程的执行结果

* 有一个结果需要从一个线程传递到另一个线程，让它们关联同一个 GuardedObject
* 如果有结果不断从一个线程到另一个线程那么可以使用消息队列（见生产者/消费者）
* JDK 中，join 的实现、Future 的实现，采用的就是此模式

![1770280653430](image/并发编程知识/1770280653430.png)

```java
public static void main(String[] args) {
    GuardedObject object = new GuardedObjectV2();
    new Thread(() -> {
        sleep(1);
        object.complete(Arrays.asList("a", "b", "c"));
    }).start();
  
    Object response = object.get(2500);
    if (response != null) {
        log.debug("get response: [{}] lines", ((List<String>) response).size());
    } else {
        log.debug("can't get response");
    }
}

class GuardedObject {
    private Object response;
    private final Object lock = new Object();

    //获取结果
    //timeout :最大等待时间
    public Object get(long millis) {
        synchronized (lock) {
            // 1) 记录最初时间
            long begin = System.currentTimeMillis();
            // 2) 已经经历的时间
            long timePassed = 0;
            while (response == null) {
                // 4) 假设 millis 是 1000，结果在 400 时唤醒了，那么还有 600 要等
                long waitTime = millis - timePassed;
                log.debug("waitTime: {}", waitTime);
                //经历时间超过最大等待时间退出循环
                if (waitTime <= 0) {
                    log.debug("break...");
                    break;
                }
                try {
                    lock.wait(waitTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 3) 如果提前被唤醒，这时已经经历的时间假设为 400
                timePassed = System.currentTimeMillis() - begin;
                log.debug("timePassed: {}, object is null {}",
                        timePassed, response == null);
            }
            return response;
        }
    }

    //产生结果
    public void complete(Object response) {
        synchronized (lock) {
            // 条件满足，通知等待线程
            this.response = response;
            log.debug("notify...");
            lock.notifyAll();
        }
    }
}
```

**6.9.2 多任务版**

* 多任务情况下，在多个类之间使用 GuardedObject 对象传参不方便，因此设计中间类，解耦等待和实现

![1770282630077](image/并发编程知识/1770282630077.png)

```java
public static void main(String[] args) throws InterruptedException {
    for (int i = 0; i < 3; i++) {
        new People().start();
    }
    Thread.sleep(1000);
    for (Integer id : Mailboxes.getIds()) {
        new Postman(id, id + "号快递到了").start();
    }
}

@Slf4j(topic = "c.People")
class People extends Thread{
    @Override
    public void run() {
        // 收信
        GuardedObject guardedObject = Mailboxes.createGuardedObject();
        log.debug("开始收信i d:{}", guardedObject.getId());
        Object mail = guardedObject.get(5000);
        log.debug("收到信id:{}，内容:{}", guardedObject.getId(),mail);
    }
}

class Postman extends Thread{
    private int id;
    private String mail;
    //构造方法
    @Override
    public void run() {
        GuardedObject guardedObject = Mailboxes.getGuardedObject(id);
        log.debug("开始送信i d:{}，内容:{}", guardedObject.getId(),mail);
        guardedObject.complete(mail);
    }
}

class  Mailboxes {
    private static Map<Integer, GuardedObject> boxes = new Hashtable<>();
    private static int id = 1;

    //产生唯一的id
    private static synchronized int generateId() {
        return id++;
    }

    public static GuardedObject getGuardedObject(int id) {
        return boxes.remove(id);
    }

    public static GuardedObject createGuardedObject() {
        GuardedObject go = new GuardedObject(generateId());
        boxes.put(go.getId(), go);
        return go;
    }

    public static Set<Integer> getIds() {
        return boxes.keySet();
    }
}
class GuardedObject {
    //标识，Guarded Object
    private int id;//添加get set方法
}
```

**6.9.3 生产者/消费者**

* 消费队列可以用来平衡生产和消费的线程资源，不需要产生结果和消费结果的线程一一对应
* 生产者仅负责产生结果数据，不关心数据该如何处理，而消费者专心处理结果数据
* 消息队列是有容量限制的，满时不会再加入数据，空时不会再消耗数据
* JDK 中各种阻塞队列，采用的就是这种模式

![1770297912179](image/并发编程知识/1770297912179.png)

```java
 public class demo {
    public static void main(String[] args) {
        MessageQueue queue = new MessageQueue(2);
        for (int i = 0; i < 3; i++) {
            int id = i;
            new Thread(() -> {
                queue.put(new Message(id,"值"+id));
            }, "生产者" + i).start();
        }
  
        new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(1000);
                    Message message = queue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"消费者").start();
    }
}

//消息队列类，Java间线程之间通信
class MessageQueue {
    private LinkedList<Message> list = new LinkedList<>();//消息的队列集合
    private int capacity;//队列容量
    public MessageQueue(int capacity) {
        this.capacity = capacity;
    }

    //获取消息
    public Message take() {
        //检查队列是否为空
        synchronized (list) {
            while (list.isEmpty()) {
                try {
                    sout(Thread.currentThread().getName() + ":队列为空，消费者线程等待");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //从队列的头部获取消息返回
            Message message = list.removeFirst();
            sout(Thread.currentThread().getName() + "：已消费消息--" + message);
            list.notifyAll();
            return message;
        }
    }

    //存入消息
    public void put(Message message) {
        synchronized (list) {
            //检查队列是否满
            while (list.size() == capacity) {
                try {
                    sout(Thread.currentThread().getName()+":队列为已满，生产者线程等待");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            //将消息加入队列尾部
            list.addLast(message);
            sout(Thread.currentThread().getName() + ":已生产消息--" + message);
            list.notifyAll();
        }
    }
}

final class Message {
    private int id;
    private Object value;
   //getter setter
}
```

### 6.10 Park & Unpark

**6.10.1 介绍和使用**

LockSupport 是用来创建锁和其他同步类的**线程原语**

LockSupport 类方法：

* `LockSupport.park()`：暂停当前线程，挂起原语
* `LockSupport.unpark(暂停的线程对象)`：恢复某个线程的运行

```java
public static void main(String[] args) {
    Thread t1 = new Thread(() -> {
        System.out.println("start...");	//1
		Thread.sleep(1000);// Thread.sleep(3000)
        // 先 park 再 unpark 和先 unpark 再 park 效果一样，都会直接恢复线程的运行
        System.out.println("park...");	//2
        LockSupport.park();
        System.out.println("resume...");//4
    },"t1");
    t1.start();
   	Thread.sleep(2000);
    System.out.println("unpark...");	//3
    LockSupport.unpark(t1);
}
```

**LockSupport 出现是为了增强 wait & notify 的功能：**

* **wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park、unpark 不需要**
* park & unpark **以线程为单位**来阻塞和唤醒线程，而 notify 只能随机唤醒一个等待线程，notifyAll 是唤醒所有等待线程
* **park & unpark 可以先 unpark，而 wait & notify 不能先 notify。类比生产消费，先消费发现有产品就消费，没有就等待；先生产就直接产生商品，然后线程直接消费**
* wait 会释放锁资源进入等待队列，**park 不会释放锁资源**，只负责阻塞当前线程，会释放 CPU

**6.10.2 原理**

* 先 park()：
  1. 当前线程调用 Unsafe.park() 方法
  2. 检查 _counter ，本情况为 0，这时获得 _mutex 互斥锁
  3. 线程进入 _cond 条件变量挂起，设置 _counter =  0
  4. 调用 Unsafe.unpark(Thread_0) 方法，设置 _counter = 1
  5. 唤醒 _cond 条件变量中的 Thread_0，Thread_0 恢复运行，设置 _counter 为 0

![1770367162608](image/并发编程知识/1770367162608.png)

* 先 unpark()：
  1. 调用 Unsafe.unpark(Thread_0) 方法，设置 _counter 为 1
  2. 当前线程调用 Unsafe.park() 方法
  3. 检查 _counter ，本情况为 1，这时线程无需挂起，继续运行，设置 _counter 为 0

![1770367225888](image/并发编程知识/1770367225888.png)

### 6.11 多把锁

多把不相干的锁：一间大屋子有两个功能睡觉、学习，互不相干。现在一人要学习，一人要睡觉，如果只用一间屋子（一个对象锁）的话，那么并发度很低

将锁的粒度细分：

* 好处，是可以增强并发度
* 坏处，如果一个线程需要同时获得多把锁，就容易发生死锁

解决方法：准备多个对象锁

```java
public static void main(String[] args) {
    BigRoom bigRoom = new BigRoom();
    new Thread(() -> { bigRoom.study(); }).start();
    new Thread(() -> { bigRoom.sleep(); }).start();
}
class BigRoom {
    private final Object studyRoom = new Object();
    private final Object sleepRoom = new Object();

    public void sleep() throws InterruptedException {
        synchronized (sleepRoom) {
            System.out.println("sleeping 2 小时");
            Thread.sleep(2000);
        }
    }

    public void study() throws InterruptedException {
        synchronized (studyRoom) {
            System.out.println("study 1 小时");
            Thread.sleep(1000);
        }
    }
}
```
