# 先来了解一下什么是时间轮

时间轮这个技术其实出来很久了，在kafka、zookeeper等技术中都有时间轮使用的方式。我第一次听这个概念，是当时我一个朋友在拼多多，负责整体架构设计时需要考虑到**超时订单的自动关单**，而订单交易量又特别多，直接去轮询数据的效率有点低，所以当时沟通下来聊到了时间轮这个东西。什么是时间轮呢？

简单来说： **时间轮是一种高效利用线程资源进行批量化调度的一种调度模型**。把大批量的调度任务全部绑定到同一个调度器上，使用这一个调度器来进行所有任务的管理、触发、以及运行。

所以时间轮的模型能够高效管理各种延时任务、周期任务、通知任务。 以后大家在工作中遇到类似的功能，可以采用时间轮机制。

如图3-11，时间轮，从图片上来看，就和手表的表圈是一样，所以称为时间轮，是因为它是以时间作为刻度组成的一个环形队列，这个环形队列采用数组来实现，数组的每个元素称为槽，每个槽可以放一个定时任务列表，叫HashedWheelBucket，它是一个双向链表，量表的每一项表示一个定时任务项（HashedWhellTimeout），其中封装了真正的定时任务TimerTask。

时间轮是由多个时间格组成，下图中有8个时间格，每个时间格代表当前时间轮的基本时间跨度（tickDuration），其中时间轮的时间格的个数是固定的。

在下图中，有8个时间格（槽），假设每个时间格的单位为1s，那么整个时间轮走完一圈需要8s钟。每秒钟指针会沿着顺时针方向移动一个，这个单位可以设置，比如以秒为单位，可以以一小时为单位，这个单位可以代表时间精度。通过指针移动，来获得每个时间格中的任务列表，然后遍历这一个时间格中的双向链表来执行任务，以此循环。

![](./assets/时间轮-1685669256790.png)

![](./assets/Netty时间轮-1686303830062.png)

# Netty时间轮原理
在 Netty 中，时间轮的实现是通过 `HashedWheelTimer` 类来完成的，它基于 JDK 的 `ScheduledExecutorService`，但提供了更高效和精确的定时任务调度机制。`HashedWheelTimer` 使用哈希方式来组织时间轮的槽，从而实现快速的任务查找和调度。

下面是 `HashedWheelTimer` 的基本实现原理：

1. **时间轮结构**：`HashedWheelTimer` 中的时间轮被划分为一系列的槽（buckets），每个槽代表一个时间单位（比如毫秒），而槽内包含了一个任务链表。槽的数量是固定的，通常是 2 的幂次方，比如 512。

2. **哈希算法**：为了将任务分散到不同的槽中，`HashedWheelTimer` 使用了一种哈希算法，将任务的执行时间映射到具体的槽。这使得任务能够均匀地分布在时间轮中，避免了大量任务堆积在某一个槽中。

3. **时间轮的推进**：`HashedWheelTimer` 通过一个定时的周期性任务来推进时间轮，每次推进一个槽。当时间轮的一个槽被推进时，其槽内的任务会被执行。

4. **延迟任务的添加**：当添加一个延迟任务时，`HashedWheelTimer` 计算任务需要等待多少个槽才能执行，然后将任务添加到相应的槽内。如果任务的等待时间大于时间轮的一个周期，那么可能需要在多个槽上注册同一个任务。

5. **任务执行**：当时间轮的槽被推进时，对应的槽内的任务会被执行。`HashedWheelTimer` 会遍历任务链表，依次执行其中的任务。执行完毕后，任务链表会被清空。

6. **任务取消**：`HashedWheelTimer` 支持取消已添加但尚未执行的任务。取消任务时，只需要从对应的槽中移除该任务即可。

总体来说，`HashedWheelTimer` 通过哈希方式将任务分布到时间轮的槽中，从而实现了高效的定时任务调度。这个机制可以减少线程切换的开销，以及避免了传统定时器中的堆积问题。值得注意的是，实际的 Netty 源代码可能在细节和性能上有所调整，因此如果需要深入了解，可以查阅 Netty 源码和文档。

# Netty时间轮的优势

`HashedWheelTimer` 是 Netty 框架中提供的一个基于时间轮算法的定时器实现,具有一些优势：

1. 高性能：`HashedWheelTimer` 在设计上采用了哈希表来存储时间轮槽，通过哈希函数将任务均匀地分布在不同的槽中。这种设计可以使得定时器的性能更好，即使在有大量任务的情况下，仍能保持较低的延迟和高效的任务执行。
2. 线程安全：`HashedWheelTimer` 在内部实现中使用了各种并发数据结构和同步机制，保证了在多线程环境下的线程安全性。
3. 高可靠性：`HashedWheelTimer` 具备任务调度的高可靠性。它在内部使用了堆外内存进行任务管理，即使在应用程序异常退出或内存回收的情况下，已经加入时间轮但尚未执行的任务也能得到恢复和执行。
4. 高度可配置：`HashedWheelTimer` 提供了丰富的配置选项，允许你根据具体需求调整定时器的精度、时间轮槽的数量、任务执行线程池等参数。
5. 功能丰富：除了基本的定时任务执行，`HashedWheelTimer` 还支持延迟任务和周期性任务的调度。

总之，Netty 的 `HashedWheelTimer` 是一个经过优化和高度可配置的时间轮实现，具备高性能、高可靠性和线程安全等优势。如果你使用 Netty 框架或需要一个高性能的定时器，那么 `HashedWheelTimer` 是一个很好的选择。

# 下面开源的组件都用到了HashedWheelTimer

以下是一些开源项目中使用了 Netty 的 `HashedWheelTimer` 组件的例子：

1. Elasticsearch：Elasticsearch 是一个开源的分布式搜索和分析引擎，它在内部使用了 Netty 的 `HashedWheelTimer` 来进行任务调度和超时处理。
2. RocketMQ：RocketMQ 是一个开源的分布式消息队列系统，它使用了 Netty 的 `HashedWheelTimer` 来处理定时任务和超时任务。
3. Dubbo：Dubbo 是一个高性能的 Java RPC 框架，它使用了 Netty 的 `HashedWheelTimer` 来进行定时任务调度和超时处理。
4. Apache Cassandra：Cassandra 是一个开源的分布式 NoSQL 数据库系统，它在内部使用了 Netty 的 `HashedWheelTimer` 来进行定时任务和超时任务的处理。

这只是一小部分使用了 Netty 的 `HashedWheelTimer` 的开源项目示例，实际上，`HashedWheelTimer` 在很多需要高性能定时任务调度的系统中被广泛使用。由于 Netty 的高性能和可靠性，它成为许多开源项目中的首选框架之一。如果你对特定项目是否使用了 `HashedWheelTimer` 感兴趣，可以进一步探索该项目的源代码或文档，以了解更多细节。

# 时间轮的使用

这里使用的时间轮是Netty这个包中提供的，使用方法比较简单。

* 先构建一个HashedWheelTimer时间轮。
  * tickDuration： 100 ，表示每个时间格代表当前时间轮的基本时间跨度，这里是100ms，也就是指针100ms跳动一次，每次跳动一个窗格
  * ticksPerWheel：1024，表示时间轮上一共有多少个窗格，分配的窗格越多，占用内存空间就越大
  * leakDetection：是否开启内存泄漏检测。
  * maxPendingTimeouts[可选参数]，最大允许等待的任务数，默认没有限制。
* 通过newTimeout()把需要延迟执行的任务添加到时间轮中

```java
@RestController
public class RedissonController { 
   @Autowired  
   RedissonClient redissonClient;  
   HashedWheelTimer hashedWheelTimer= new HashedWheelTimer(new DefaultThreadFactory("demo-timer"), 100, TimeUnit.MILLISECONDS, 1024, false);
 /**   
 * 添加延迟任务   
 * @param delay   
 */ 
@GetMapping("/{delay}")  
public void tick(@PathVariable("delay")Long delay){
       System.out.println("currentDate:"+new Date());  
       hashedWheelTimer.newTimeout(timeout -> {  
       System.out.println("executeDate:"+new Date());  
       }, delay, TimeUnit.SECONDS);   
   }
  }
```

# 时间轮的原理解析

时间轮的整体原理，分为几个部分。

* 创建时间轮
  时间轮本质上是一个环状数组，比如我们初始化时间轮时：ticksPerWheel=8，那么意味着这个环状数组的长度是8，如图3-12所示。

```java
HashedWheelBucket[] wheel = new HashedWheelBucket[ticksPerWheel];
```

![](./assets/时间轮-1685669386990.png)

* 添加任务，如图3-13所示

  * 当通过newTimeout()方法添加一个延迟任务时，该任务首先会加入到一个阻塞队列中中。
  * 然后会有一个定时任务从该队列获取任务，添加到时间轮的指定位置，计算方法如下。
  * 当前任务的开始执行时间除以每个窗口的时间间隔，得到一个calculated值

![](./assets/时间轮-1685669649291.png)

* 任务执行
  Worker线程按照每次间隔时间转动后，得到该时间窗格中的任务链表，然后从链表的head开始逐个取出任务，有两个判断条件
  * 当前任务需要转动的圈数为0，表示任务是当前圈开始执行
  * 当前任务达到了delay时间，也就是timeout.deadline <= deadline
  * 最终调用timeout.expire()方法执行任务。

```java
public void expireTimeouts(long deadline) {   
 HashedWheelTimeout timeout = head;  
// process all timeouts  
while (timeout != null) {   
 HashedWheelTimeout next = timeout.next;  
if (timeout.remainingRounds <= 0) {   
 next = remove(timeout);   
 if (timeout.deadline <= deadline) {        
    timeout.expire();  
  } else { 
    // The timeout was placed into a wrong slot. This should never happen.
    throw new IllegalStateException(String.format("timeout.deadline (%d) > deadline (%d)", timeout.deadline, deadline));   
 } 
} else if (timeout.isCancelled()) {
          next = remove(timeout);   
 } else {    
   timeout.remainingRounds --;   
 } 
  timeout = next;  
 }
}
```

# 时间轮的源码分析

## HashedWheelTimer的构造

* 调用createWheel创建一个时间轮，时间轮数组一定是2的幂次方，比如传入的ticksPerWheel=6，那么初始化的wheel长度一定是8，这样是便于时间格的计算。
* tickDuration，表示时间轮的跨度，代表每个时间格的时间精度，以纳秒的方式来表现。
* 把工作线程Worker封装成WorkerThread，从名字可以知道，它就是最终那个负责干活的线程。
* 因为一圈的长度为2的n次方，mask = 2^n-1后低位将全部是1，然后deadline&mask == deadline%wheel.length

```java
public HashedWheelTimer(ThreadFactory threadFactory,long tickDuration, TimeUnit unit, int ticksPerWheel,long maxPendingTimeouts) { 
// 创建时间轮基本的数据结构，一个数组。长度为不小于ticksPerWheel的最小2的n次方
wheel = createWheel(ticksPerWheel);  
// 这是一个标示符，用来快速计算任务应该呆的格子。  
// 我们知道，给定一个deadline的定时任务，其应该呆的格子=deadline%wheel.length.但是%操作是个相对耗时的操作，所以使用一种变通的位运算代替：  
// 因为一圈的长度为2的n次方，mask = 2^n-1后低位将全部是1，然后deadline&mask == deadline%wheel.length  
// java中的HashMap在进行hash之后，进行index的hash寻址寻址的算法也是和这个一样的  
mask = wheel.length - 1;  
//时间轮的基本时间跨度，（tickDuration传入是1的话，这里会转换成1000000）   
this.tickDuration = unit.toNanos(tickDuration);   
// 校验是否存在溢出。即指针转动的时间间隔不能太长而导致tickDuration*wheel.length>Long.MAX_VALUE  
if (this.tickDuration >= Long.MAX_VALUE / wheel.length) {  
   throw new IllegalArgumentException(
           String.format("tickDuration: %d (expected: 0 < tickDuration in nanos < %d",tickDuration, Long.MAX_VALUE / wheel.length));  
  }  
//把worker包装成thread  
workerThread = threadFactory.newThread(worker);  
this.maxPendingTimeouts = maxPendingTimeouts;  
//如果HashedWheelTimer实例太多，那么就会打印一个error日志   
 if (INSTANCE_COUNTER.incrementAndGet() > INSTANCE_COUNT_LIMIT &&        WARNED_TOO_MANY_INSTANCES.compareAndSet(false, true)) {   
   reportTooManyInstances();   
 }
}
```

* 对传入的ticksPerWheel进行整形
* 初始化固定长度的HashedWheelBucket

```java
private static HashedWheelBucket[] createWheel(int ticksPerWheel) { 
   if (ticksPerWheel <= 0) {  
       throw new IllegalArgumentException("ticksPerWheel must be greater than 0: " + ticksPerWheel); 
      }   
     if (ticksPerWheel > 1073741824) {   
          throw new IllegalArgumentException("ticksPerWheel may not be greater than 2^30: " + ticksPerWheel);  
      }  
    //对传入的时间轮大小进行整形，整形乘以2的幂次方  
    ticksPerWheel = normalizeTicksPerWheel(ticksPerWheel);  
    //初始化一个固定长度的Bucket数组  
    HashedWheelBucket[] wheel = new HashedWheelBucket[ticksPerWheel];   
     for (int i = 0; i < wheel.length; i++) {  
       wheel[i] = new HashedWheelBucket();  
      }  
       return wheel;
           }
```

## 添加任务到时间轮

完成时间轮的初始化之后，并没有去启动时间轮，继续看FailbackClusterInvoker中的代码。
构建了一个RetryTimerTask，也就是一个重试的定时任务，接着把这个任务通过newTimeout加入到时间轮中，其中

* retryTimerTask，表示具体的重试任务
* RETRY_FAILED_PERIOD ， 表示重试间隔时间，默认为5s

```java
RetryTimerTask retryTimerTask = new RetryTimerTask(loadbalance, invocation, invokers, lastInvoker, retries, RETRY_FAILED_PERIOD);
failTimer.newTimeout(retryTimerTask, RETRY_FAILED_PERIOD, TimeUnit.SECONDS);
```

调用newTimeout方法，把任务添加进来。

```java
public Timeout newTimeout(TimerTask task, long delay, TimeUnit unit) {  
  if (task == null) {  
  throw new NullPointerException("task");  
  }   
 if (unit == null) {   
     throw new NullPointerException("unit");  
  }   
 //统计任务个数 
   long pendingTimeoutsCount = pendingTimeouts.incrementAndGet();   
 //判断最大任务数量是否超过限制  
if (maxPendingTimeouts > 0 && pendingTimeoutsCount > maxPendingTimeouts) {
    pendingTimeouts.decrementAndGet();  
      throw new RejectedExecutionException(
              "Number of pending timeouts ("+ pendingTimeoutsCount + ") is greater than or equal to maximum allowed pending "                                             + "timeouts (" + maxPendingTimeouts + ")");  
  }  
  //如果时间轮没有启动，则通过start方法进行启动  
  start();  
// Add the timeout to the timeout queue which will be processed on the next tick.  
// During processing all the queued HashedWheelTimeouts will be added to the correct HashedWheelBucket.  
//计算任务的延迟时间，通过当前的时间+当前任务执行的延迟时间-时间轮启动的时间。   
 long deadline = System.nanoTime() + unit.toNanos(delay) - startTime;  
//在delay为正数的情况下，deadline是不可能为负数  
//如果为负数，那么说明超过了long的最大值   
 if (delay > 0 && deadline < 0) {   
     deadline = Long.MAX_VALUE;    }  
  //创建一个Timeout任务，理论上来说，这个任务应该要加入到时间轮的时间格子中，但是这里并不是先添加到时间格，而是先  
//加入到一个阻塞队列，然后等到时间轮执行到下一个格子时，再从队列中取出最多100000个任务添加到指定的时间格（槽）中。  
 HashedWheelTimeout timeout = new HashedWheelTimeout(this, task, deadline);
 timeouts.add(timeout);
 return timeout;
}
```

**start**
任务添加到阻塞队列之后，我们再来看启动方法
start方法会根据当前的workerState状态来启动时间轮。并且用了startTimeInitialized来控制线程的运行，如果workerThread没有启动起来，那么newTimeout方法会一直阻塞在运行start方法中。如果不阻塞，newTimeout方法会获取不到startTime。

```java
public void start() {   
 //workerState一开始的时候是0（WORKER_STATE_INIT），然后才会设置为1（WORKER_STATE_STARTED）  
switch (WORKER_STATE_UPDATER.get(this)) {  
    case WORKER_STATE_INIT:  
      if (WORKER_STATE_UPDATER.compareAndSet(this, WORKER_STATE_INIT, WORKER_STATE_STARTED)) {   
           workerThread.start();   
       }      
      break;   
     case WORKER_STATE_STARTED:
        break;
     case WORKER_STATE_SHUTDOWN:  
        throw new IllegalStateException("cannot be started once stopped");
     default:
        throw new Error("Invalid WorkerState");   
 }  
  // 等待worker线程初始化时间轮的启动时间   
 while (startTime == 0) {  
  try {   
         //这里使用countDownLauch来确保调度的线程已经被启动     
         startTimeInitialized.await();   
     } catch (InterruptedException ignore) {
      // Ignore - it will be ready very soon.
        }   
        }
 }
}
```

**启动时间轮**
调用start（）方法， 会调用workerThread.start();来启动一个工作线程，这个工作线程是在构造方法中初始化的，包装的是一个Worker内部线程类。
所以直接进入到Worker这个类的run方法，了解下它的设计逻辑

```java
public void run() {  
  // 初始化startTime，表示时间轮的启动时间 
   startTime = System.nanoTime(); 
   if (startTime == 0) {   
     // We use 0 as an indicator for the uninitialized value here, so make sure it's not 0 when initialized.   
     startTime = 1; 
   }     // 唤醒被阻塞的start()方法。   
   startTimeInitialized.countDown();   
   do {        //返回每tick一次的时间间隔  
  final long deadline = waitForNextTick(); 
  if (deadline > 0) {   
 //计算时间轮的槽位  
 int idx = (int) (tick & mask);  
 //移除掉CancelledTask      
  processCancelledTasks();   
 //得到当前指针位置的时间槽   
 HashedWheelBucket bucket =wheel[idx];  
 //将newTimeout()方法中加入到待处理定时任务队列中的任务加入到指定的格中
 transferTimeoutsToBuckets();     
 //运行目前指针指向的槽中的bucket链表中的任务      
 bucket.expireTimeouts(deadline);      
 tick++;   
  }   
 } while (WORKER_STATE_UPDATER.get(HashedWheelTimer.this) == WORKER_STATE_STARTED);
   //如果Worker_State一只是started状态，就一直循环   
 // Fill the unprocessedTimeouts so we can return them from stop() method. 
   for (HashedWheelBucket bucket : wheel) {
           bucket.clearTimeouts(unprocessedTimeouts); 
//清除时间轮中不需要处理的任务  
}  
  for (; ; ) {  
 //遍历任务队列，发现如果有任务被取消，则添加到unprocessedTimeouts,也就是不需要处理的队列中。  
 HashedWheelTimeout timeout = timeouts.poll();   
 if (timeout == null) {     
 break;        }   
 if (!timeout.isCancelled()) {   
         unprocessedTimeouts.add(timeout);  
  }
    }    //处理被取消的任务.   
 processCancelledTasks();
}
```

## 时间轮指针跳动

这个方法的主要作用就是返回下一个指针指向的时间间隔，然后进行sleep操作。
大家可以想象一下，一个钟表上秒与秒之间是有时间间隔的，那么waitForNextTick就是根据当前时间计算出跳动到下个时间的时间间隔，然后进行sleep，然后再返回当前时间距离时间轮启动时间的时间间隔。
说得再直白一点：假设当前的tickDuration的间隔是1s，tick默认=0， 此时第一次进来，得到的deadline=1，也就是下一次跳动的时间间隔是1s。假设当前处于

```java
private long waitForNextTick() {  
//tick表示总的tick数  
//tickDuration表示每个时间格的跨度，所以deadline返回的是下一次时间轮指针跳动的时间    long deadline = tickDuration * (tick + 1);    for (; ; ) {   
 //计算当前时间距离启动时间的时间间隔   
 final long currentTime = System.nanoTime() - startTime;   
 //通过下一次指针跳动的延迟时间距离当前时间的差额，这个作为sleep时间使用。   
 // 其实线程是以睡眠一定的时候再来执行下一个ticket的任务的  
long sleepTimeMs = (deadline - currentTime + 999999) / 1000000;   
 //sleepTimeMs小于零表示走到了下一个时间槽位置   
 if (sleepTimeMs <= 0) {   
       if (currentTime == Long.MIN_VALUE) {  
              return -Long.MAX_VALUE;  
      } else {      
    return currentTime;   
       }  
      } 
       if (isWindows()) {   
    sleepTimeMs = sleepTimeMs / 10 * 10;   
     }  
      //进入到这里进行sleep，表示当前时间距离下一次tick时间还有一段距离，需要sleep。
     try {   
       Thread.sleep(sleepTimeMs);   
     } catch (InterruptedException ignored) {  
        if (WORKER_STATE_UPDATER.get(HashedWheelTimer.this) == WORKER_STATE_SHUTDOWN) {
            return Long.MIN_VALUE;  
          }  
      }
    }
}
```

## transferTimeoutsToBuckets

转移任务到时间轮中，前面我们讲过，任务添加进来时，是先放入到阻塞队列。
而在现在这个方法中，就是把阻塞队列中的数据转移到时间轮的指定位置。
在这个转移方法中，写死了一个循环，每次都只转移10万个任务。
然后根据HashedWheelTimeout的deadline延迟时间计算出时间轮需要运行多少次才能运行当前的任务，如果当前的任务延迟时间大于时间轮跑一圈所需要的时间，那么就计算需要跑几圈才能到这个任务运行。
最后计算出该任务在时间轮中的槽位，添加到时间轮的链表中。

```java
private void transferTimeoutsToBuckets() {   
 // 循环100000次，也就是每次转移10w个任务 
   for (int i = 0; i < 100000; i++) {   
     //从阻塞队列中获得具体的任务  
    HashedWheelTimeout timeout = timeouts.poll();  
    if (timeout == null) {   
       // all processed   
       break;   
   }
if (timeout.state() == HashedWheelTimeout.ST_CANCELLED) {
  // Was cancelled in the meantime.     
 continue;   
 }   
//计算tick次数，deadline表示当前任务的延迟时间，tickDuration表示时间槽的间隔，两者相除就可以计算当前任务需要tick几次才能被执行  
long calculated = timeout.deadline / tickDuration;   
// 计算剩余的轮数, 只有 timer 走够轮数, 并且到达了 task 所在的 slot, task 才会过期.(被执行)  
timeout.remainingRounds = (calculated - tick) / wheel.length;   
 //如果任务在timeouts队列里面放久了, 以至于已经过了执行时间, 这个时候就使用当前tick, 也就是放到当前bucket, 此方法调用完后就会被执行  
final long ticks = Math.max(calculated, tick);  
// 算出任务应该插入的 wheel 的 slot, stopIndex = tick 次数 & mask, mask = wheel.length - 1  
int stopIndex = (int) (ticks & mask);  
//把timeout任务插入到指定的bucket链中。   
 HashedWheelBucket bucket = wheel[stopIndex];  
 bucket.addTimeout(timeout);  
}
}
```

## 运行时间轮中的任务

当指针跳动到某一个时间槽中时，会就触发这个槽中的任务的执行。该功能是通过expireTimeouts来实现
这个方法的主要作用是： 过期并执行格子中到期的任务。也就是当tick进入到指定格子时，worker线程会调用这个方法
HashedWheelBucket是一个链表，所以我们需要从head节点往下进行遍历。如果链表没有遍历到链表尾部那么就继续往下遍历。
获取的timeout节点节点，如果剩余轮数remainingRounds大于0，那么就说明要到下一圈才能运行，所以将剩余轮数减一；
如果当前剩余轮数小于等于零了，那么就将当前节点从bucket链表中移除，并判断一下当前的时间是否大于timeout的延迟时间，如果是则调用timeout的expire执行任务。

```java
void expireTimeouts(long deadline) {   
 HashedWheelTimeout timeout = head;   
 // 遍历当前时间槽中的所有任务 
   while (timeout != null) {  
      HashedWheelTimeout next = timeout.next;  
    //如果当前任务要被执行，那么remainingRounds应该小于或者等于0  
    if (timeout.remainingRounds <= 0) {  
    //从bucket链表中移除当前timeout，并返回链表中下一个timeout   
       next = remove(timeout);   
       //如果timeout的时间小于当前的时间，那么就调用expire执行task   
     if (timeout.deadline <= deadline) {   
           timeout.expire();  
        } else {
         //不可能发生的情况，就是说round已经为0了，deadline却>当前槽的deadline 
        // The timeout was placed into a wrong slot. This should never happen.         
    throw new IllegalStateException(
        String.format("timeout.deadline (%d) > deadline (%d)", timeout.deadline, deadline));      
}
 } else if (timeout.isCancelled()) { 
           next = remove(timeout);
        } else {   
         //因为当前的槽位已经过了，说明已经走了一圈了，把轮数减一
        timeout.remainingRounds--;  
    }  
  //把指针放置到下一个timeout   
     timeout = next; 
   }}
```
