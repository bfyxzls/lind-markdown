# 组件介绍

Google Guava是一个Java开发库，提供了许多实用的工具和数据结构，可以简化Java编程过程。以下是一些Guava组件的使用实例

1. 集合工具（Collections Utilities）：
   Guava提供了许多用于处理集合的实用工具，如过滤、转换、拆分等。例如，使用`Lists.newArrayList()`创建一个新的ArrayList，使用`Iterables.filter()`过滤集合中的元素，使用`Iterables.transform()`转换集合中的元素等。

```java
import com.google.common.collect.Lists;
import com.google.common.collect.Iterables;

List<Integer> numbers = Lists.newArrayList(1, 2, 3, 4, 5);

Iterable<Integer> filteredNumbers = Iterables.filter(numbers, n -> n % 2 == 0);
System.out.println("Filtered numbers: " + filteredNumbers); // 输出：Filtered numbers: [2, 4]

Iterable<String> transformedNumbers = Iterables.transform(numbers, n -> "Number: " + n);
System.out.println("Transformed numbers: " + transformedNumbers); // 输出：Transformed numbers: [Number: 1, Number: 2, Number: 3, Number: 4, Number: 5]
```

2. 字符串工具（Strings Utilities）：
   Guava提供了一些用于处理字符串的实用工具方法，如判空、拆分、连接等。例如，使用`Strings.isNullOrEmpty()`检查字符串是否为空或null，使用`Splitter`拆分字符串，使用`Joiner`连接字符串等。

```java
import com.google.common.base.Strings;
import com.google.common.base.Splitter;
import com.google.common.base.Joiner;

String str = "Hello,World,Guava";

boolean isNullOrEmpty = Strings.isNullOrEmpty(str);
System.out.println("Is string null or empty? " + isNullOrEmpty); // 输出：Is string null or empty? false

Iterable<String> splitStrings = Splitter.on(',').split(str);
System.out.println("Split strings: " + splitStrings); // 输出：Split strings: [Hello, World, Guava]

String joinedString = Joiner.on('-').join(splitStrings);
System.out.println("Joined string: " + joinedString); // 输出：Joined string: Hello-World-Guava
```

3. 缓存工具（Caching Utilities）：
   Guava的缓存工具类提供了一种简单的方法来创建缓存并自定义其行为，如缓存大小、过期时间等。以下是一个简单的示例：

```java
import com.google.common.cache.Cache;
import com.google.common.cache.CacheBuilder;

Cache<String, Integer> cache = CacheBuilder.newBuilder()
        .maximumSize(100)
        .build();

cache.put("key1", 10);
cache.put("key2", 20);

int value1 = cache.getIfPresent("key1");
System.out.println("Value 1: " + value1); // 输出：Value 1: 10

int value2 = cache.getIfPresent("key2");
System.out.println("Value 2: " + value2); // 输出：Value 2: 20
```

这只是Guava库中一些组件的使用实例，该库还提供了许多其他实用的

工具类和数据结构，如函数式编程支持、文件操作、并发工具等。您可以查阅Guava官方文档以了解更多详细信息：[Guava官方文档](https://github.com/google/guava/wiki)。

# 为什么ImmutableMap是不可变的

ImmutableMap是Google Guava库中的一个不可变（immutable）映射（Map）实现。它被设计为主要用于读取操作，因为它在构建之后不可修改，从而提供了一些性能和线程安全方面的优势。

实现原理：
ImmutableMap的实现基于一种数据结构称为Trie（前缀树）。它使用一个树状结构来存储键值对，其中每个节点代表一个键的字符，从根节点到叶子节点的路径表示一个完整的键。这种数据结构使得ImmutableMap具有高效的查询和查找特性。

如何实现：
在Guava库中，ImmutableMap的实现通过静态工厂方法或构建器来创建。以下是两种常见的创建ImmutableMap的方式：

1. 使用静态工厂方法：

```java
import com.google.common.collect.ImmutableMap;

ImmutableMap<String, Integer> map = ImmutableMap.of(
    "key1", 1,
    "key2", 2,
    "key3", 3
);
```

2. 使用构建器：

```java
import com.google.common.collect.ImmutableMap;

ImmutableMap<String, Integer> map = ImmutableMap.<String, Integer>builder()
    .put("key1", 1)
    .put("key2", 2)
    .put("key3", 3)
    .build();
```

无论使用哪种方式创建ImmutableMap，一旦构建完成，它就不可修改。如果尝试对ImmutableMap进行修改操作（如添加、删除或更新键值对），将会抛出UnsupportedOperationException异常。

由于ImmutableMap是不可变的，它具有以下优势：

1. 线程安全：由于ImmutableMap不可修改，多个线程可以同时读取它，而无需进行额外的同步措施。这提供了线程安全性，避免了在并发环境中出现数据竞争和不一致性。
2. 性能优化：ImmutableMap的不可变性使得它可以进行一些内部优化，如在构建时进行预计算和缓存结果。这提高了查询和查找操作的性能。
3. 可靠性和可预测性：ImmutableMap的不可变性确保了其在创建后保持不变，不会受到外部因素的影响。这使得它更加可靠，并且在多线程环境下更易于推理和调试。

需要注意的是，如果需要对映射进行频繁的修改操作，那么ImmutableMap可能不是最佳选择，因为每次修改都会创建一个新的ImmutableMap实例。在这种情况下，应该考虑使用可变的Map实现，如HashMap。

总结：ImmutableMap是一种不可变的映射实现，它在构建之后不可修改，提供了线程安全、性能优化和可靠性等优势。它的实现基于Trie数据

结构，并通过静态工厂方法或构建器来创建。
