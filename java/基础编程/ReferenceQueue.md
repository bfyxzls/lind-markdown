# 概念

软引用、弱引用和虚引用是 Java 中的三种引用类型，它们提供了对对象生命周期的不同控制方式。下面是对这三种引用类型的说明：

1. 软引用（Soft Reference）：
   软引用是一种相对强引用弱化的引用类型。当内存不足时，垃圾回收器会根据对象的引用类型进行回收。如果一个对象只被软引用引用，而没有强引用引用它，那么在内存不足时，该对象就有可能被回收。软引用可以用于实现内存敏感的缓存机制，在内存不足时释放缓存对象，以避免 OutOfMemoryError 错误。
2. 弱引用（Weak Reference）：
   弱引用是一种比软引用更弱化的引用类型。只要垃圾回收器运行，无论内存是否充足，被弱引用引用的对象都有可能被回收。弱引用常用于实现辅助性数据结构，如 WeakHashMap，或者用于解决内存泄漏问题，例如缓存中的键使用弱引用，当键对象不再被强引用引用时，该键会自动被回收。
3. 虚引用（Phantom Reference）：
   虚引用是一种最弱化的引用类型。虚引用的存在几乎没有引用的意义，无法通过虚引用访问对象。它主要用于跟踪对象被垃圾回收的状态。当一个对象被虚引用引用时，在对象被垃圾回收器回收时，会将引用放入关联的 ReferenceQueue 中，以便在对象被回收时收到通知。虚引用可以用于执行某些对象回收前的清理操作，比如在对象被回收时释放相关的资源。

需要注意的是，无论是软引用、弱引用还是虚引用，它们都需要与 ReferenceQueue 结合使用，以便在对象被回收时进行相应的处理。使用不同类型的引用和相应的队列，可以实现对对象生命周期的灵活控制和监控。

# 代码说明

ReferenceQueue 是 Java 中的一个类，用于配合软引用、弱引用和虚引用等引用类型的使用。它可以用于跟踪对象是否被垃圾回收器回收，并在对象被回收时执行一些额外的操作。

使用 ReferenceQueue 的一般步骤如下：

1. 创建一个 ReferenceQueue 对象：

```java
ReferenceQueue<MyObject> referenceQueue = new ReferenceQueue<>();
```

2. 创建需要监视的引用对象：

```java
MyObject myObject = new MyObject();
Reference<MyObject> reference = new SoftReference<>(myObject, referenceQueue);
```

在这个例子中，我们创建了一个软引用 reference，并指定了要监视的对象 myObject 和 referenceQueue。

3. 在需要的时候检查 ReferenceQueue：

```java
Reference<? extends MyObject> polledRef = referenceQueue.poll();
if (polledRef != null) {
    // 引用对象被回收，执行相关操作
}
```

使用 referenceQueue.poll() 方法可以从队列中获取已经被回收的引用对象。如果队列为空，则返回 null。

注意事项：

- ReferenceQueue 的泛型类型应该与引用类型的泛型类型一致。
- 使用 ReferenceQueue 需要在对象被垃圾回收时执行一些额外操作时，如释放资源或清理缓存等。

这是 ReferenceQueue 类的基本使用方法。根据具体的需求，你还可以使用不同类型的引用（如弱引用、虚引用）和相应的队列来实现更复杂的对象监控和回收操作。
