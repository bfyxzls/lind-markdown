# █ Stream 流库

# 一. 流的创建

## 1. 根据已有数据创建

对于集合, `Collection`提供了`stream()`方法, 可以将当前集合转换成流

```java
// List 转成流
List<Integer> integerList = DemoUtil.intList();
Stream<Integer> listStream = integerList.stream();
// Set 转成流
Set<Integer> integerSet = DemoUtil.intSet();
Stream<Integer> setStream = integerSet.stream();
```

对于数组, 则可以通过 `Stream.of(...)` 或 `Arrays.stream(...)` 这两个静态方法来获取流

```java
// 引用类型数组
Integer[] array1= {1,2,3,4,5};
Stream<Integer> intStream = Stream.of(array1);

// 基本类型数组
int[] array2 = {1,2,3,4,5};
IntStream is = Arrays.stream(array2);
```

如果只想要数组中的一部分作为流的元素, 则可以使用`Arrays.stream()`的重载方法, 提供目标元素的起止下标

```java
// 截取数组中的部分数据作为流, 前闭后开
IntStream is2 = Arrays.stream(array2, 2, 5);
is2.forEach(System.out::println);
Stream<Integer> is3 = Arrays.stream(array1, 2, 5);
is3.forEach(System.out::println);
```

## 2. 无限流

`Stream` 还提供了两个创建无限流的方法

`generate(...)`, 接受一个 supplier 对象, 每次需要从流中获取元素时, 就会执行 supplier 的方法

```java
// 无限生成随机数
Stream<Double> g = Stream.generate(Math::random);
```

`iterate(...)`, 接受一个初始值以及 `UnaryOperation `对象, 会反复将方法作用到前一个元素中, 相当于递推公式

```java
// 1，2，3，4...
Stream<Integer> it = Stream.iterate(1, a -> a + 1);
```

Java 9 之后, `iterate`有一个重载方法, 增加了`predict`对象参数, 可以指定继续递推的条件, 可以用来创建有递推规律的有限流

```java
// 1，2，3，4...100
Stream<Integer> it = Stream.iterate(1, a -> a <= 100, a -> a + 1);
```



# 二. 流的处理

## 截取, 连接

`skip` 跳过开头的 m 个元素, `limit`则是取得 n 个元素后结束流, 配合使用即可截取流中的部分数据

```java
// 跳过前3个元素
Stream.of(1, 2, 3, 4, 5, 6)
    .skip(2)
    .forEach(System.out::println);

// 截取无限流的前20个元素
Stream.iterate(1, a -> a + 1)
    .limit(20)
    .forEach(System.out::println);
```

`concat`可以将两个不同的流连接起来, 合成一个流, 注意第一个流不能是无限流

```java
// 合并 1，2，3 和 4，5，6
Stream.concat(Stream.of(1,2,3), Stream.of(6,7,8))
    .forEach(System.out::println);
```

## 过滤, 变换

`distinct`

`filter`

```java

```

`map`

`peek`

`flatMap`



# 三. 约简处理

## 1. 基本约简

```java
// min
// max
// findFirst
// findAny
// anyMatch
// allMatch
// noneMatch
```

## 2. Optional

# 四. 结果收集

# 