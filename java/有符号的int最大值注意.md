在 Java 中，`int` 类型的取值范围是从 -2的31次方（-2,147,483,648）到 2的31次方减1（2,147,483,647），而不是2的32次方减1。

`int` 类型是用来表示整数的基本数据类型，占用 4 个字节（32 位）。其中一位用来表示符号位（正负号），所以有效的位数是 31 位。因此，最大值是 2的31次方减1，而不是2的32次方减1。

如果你需要表示更大范围的整数，可以使用 `long` 类型，它是占用 8 个字节（64 位）的整数类型。`long` 类型的取值范围是从 -2的63次方到 2的63次方减1。

以下是一些示例代码，展示 `int` 和 `long` 类型的取值范围：

```java
int maxInt = Integer.MAX_VALUE;
int minInt = Integer.MIN_VALUE;
System.out.println("Max int value: " + maxInt);  // 输出 2,147,483,647
System.out.println("Min int value: " + minInt);  // 输出 -2,147,483,648

long maxLong = Long.MAX_VALUE;
long minLong = Long.MIN_VALUE;
System.out.println("Max long value: " + maxLong);  // 输出 9,223,372,036,854,775,807
System.out.println("Min long value: " + minLong);  // 输出 -9,223,372,036,854,775,808
```

