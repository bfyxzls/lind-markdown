# 锁

* Lock
  * synchronized关键字
  * ReentrantLock
  * ReentrantReadWriteLock
  * CountDownLatch
  * Semaphore

# 详细介绍

在Java中，有几种不同的锁类型，每种锁都有其适用的使用场景。以下是常见的几种锁类型及其使用场景：

1. **synchronized关键字**：

   - 使用场景：`synchronized`关键字是Java内置的基本锁机制，适用于简单的线程同步场景。它可以修饰方法或代码块，并且只能通过获取对象的锁来保护临界区。当一个线程进入`synchronized`代码块或方法时，它会获取对象锁，其他线程需要等待锁释放后才能进入临界区。
   - 注意事项：`synchronized`是可重入锁，一个线程可以重复获取相同锁。
2. **ReentrantLock**：

   - 使用场景：`ReentrantLock`是Java提供的可重入锁实现，相比`synchronized`更加灵活。它提供了更多的高级功能，如可定时的、可中断的、公平或非公平的锁等。适用于复杂的线程同步场景，可以更精确地控制锁的获取和释放。
   - 注意事项：在使用`ReentrantLock`时，需要手动调用`lock()`方法获取锁，并在使用完毕后调用`unlock()`方法释放锁。
3. **ReadWriteLock**：

   - 使用场景：`ReadWriteLock`提供了读写分离的锁机制，适用于读多写少的场景。它允许多个线程同时获取读锁，但在写锁被持有时，其他线程无法获取读锁或写锁。这样可以提高并发性能，适合于对共享数据进行频繁读取的场景。
   - 注意事项：`ReadWriteLock`有两个实现类：`ReentrantReadWriteLock`是可重入的，而`StampedLock`是一种更高级的读写锁，提供了更多功能。
4. **Condition**：

   - 使用场景：`Condition`是在`ReentrantLock`中使用的，用于实现更复杂的线程同步场景。它可以让线程在某个条件满足时等待或继续执行。通常与`ReentrantLock`配合使用，通过`Condition`实例的`await()`、`signal()`和`signalAll()`方法来实现线程的等待和唤醒。
   - 注意事项：`Condition`需要与`ReentrantLock`一起使用，通过`lock.newCondition()`方法创建。
5. **Semaphore**：

   - 使用场景：`Semaphore`是一种计数信号量，用于控制同时访问某个资源的线程数量。它允许多个线程同时访问，但限制了同时访问的线程数量。适用于控制并发线程数的场景，例如限制同时访问数据库连接或线程池中
6. **CountDownLatch**

   - 使用场景：
     - 同步多个线程：CountDownLatch常用于一组线程需要等待其他线程完成某个操作后才能继续执行的场景。比如，主线程等待多个工作线程完成任务后再进行下一步操作。
     - 等待初始化完成：可以使用CountDownLatch来等待一组资源的初始化完成，确保在使用这些资源之前，它们已经被正确初始化。
   - 注意事项：
     - 初始化计数器数量：在创建CountDownLatch实例时，需要根据实际需要指定计数器的初始数量。计数器的值应该与需要等待的线程数量相匹配，否则会导致线程永远等待或无法达到预期的同步效果。
     - 适当调用countDown()：每个需要等待的线程在完成操作后，需要适时调用countDown()方法减少计数器的值。确保计数器减少到0，以便等待线程能够继续执行。
     - 调用await()等待计数器归零：需要等待的线程在适当的时机调用await()方法等待计数器归零。一旦计数器归零，等待线程将被唤醒并继续执行后续操作。
     - 注意线程安全：CountDownLatch本身是线程安全的，可以被多个线程同时使用。但是，在使用CountDownLatch时，需要注意线程安全性，确保在多线程环境下正确地进行计数器的操作。
     - 只能使用一次：CountDownLatch是一次性的，一旦计数器归零，就无法重置或再次使用。如果需要多次等待操作，可以考虑使用CyclicBarrier等其他线程同步工具。

# CountDownLatch的应用

> 在Keycloak源码中，CountDownLatch被广泛用于线程同步和等待的场景。以下是一些Keycloak中使用CountDownLatch的示例

1. **启动器等待服务器启动**：在Keycloak的启动过程中，有一个启动器类(`org.keycloak.services.util.ServerStartup`)负责启动各个子系统，并在所有子系统都成功启动后才继续执行后续操作。这里使用了一个`CountDownLatch`来实现等待子系统启动的功能。

```java
   CountDownLatch startupLatch = new CountDownLatch(numSubsystems);
   // ...
   // 在每个子系统启动成功后，调用 startupLatch.countDown();
   // ...
   startupLatch.await();
```

在启动过程中，每个子系统启动成功后都会调用`startupLatch.countDown()`方法来减少计数器。主线程使用`startupLatch.await()`方法来等待所有子系统启动完成后继续执行。
2. **测试类中的并发测试**：Keycloak的测试代码中也经常使用`CountDownLatch`来实现并发测试的同步。例如，在某个测试方法中，可以创建多个并发线程来执行相同的操作，并使用`CountDownLatch`来等待所有线程执行完毕。

```java
   CountDownLatch finishLatch = new CountDownLatch(numThreads);
   // ...
   for (int i = 0; i < numThreads; i++) {
       Thread thread = new Thread(() -> {
           // 并发操作代码
           // ...
           finishLatch.countDown();
       });
       thread.start();
   }
   // ...
   finishLatch.await();
```

在这个示例中，创建了多个并发线程执行一段并发操作的代码。每个线程执行完毕后都会调用`finishLatch.countDown()`来减少计数器。主线程使用`finishLatch.await()`等待所有线程执行完毕后继续执行后续断言或验证。

这些示例展示了在Keycloak中如何使用`CountDownLatch`实现线程同步和等待的功能。`CountDownLatch`被用于等待子系统启动、并发测试等场景，在多线程环境中起到了线程同步和等待的作用，确保各个操作按预期顺序执行。
