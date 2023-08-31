
# 限流
Redisson是一个强大的Java Redis客户端库，它提供了丰富的分布式数据结构和功能，包括限流功能。Redisson中的限流功能主要通过RateLimiter对象来实现。下面是一个简单示例，演示如何在Redisson中使用限流功能：

* 首先，确保在Maven或Gradle中添加Redisson的依赖：
```
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.16.0</version>
</dependency>

```
# 计数限流

以下是一个使用Redisson实现限流的示例代码，该示例使用Redisson的`RateLimiter`对象来限制每秒处理的请求数：

```java
import org.redisson.Redisson;
import org.redisson.api.RRateLimiter;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;

public class RedissonRateLimiterExample {
    public static void main(String[] args) {
        // 创建Redisson客户端连接
        Config config = new Config();
        config.useSingleServer().setAddress("redis://localhost:6379");
        RedissonClient redisson = Redisson.create(config);

        // 创建RRateLimiter对象，限制每秒处理3个请求
        RRateLimiter rateLimiter = redisson.getRateLimiter("myRateLimiter");
        rateLimiter.trySetRate(RateType.OVERALL, 3, 1, RateIntervalUnit.SECONDS);

        // 模拟处理请求
        for (int i = 1; i <= 5; i++) {
            if (rateLimiter.tryAcquire()) {
                System.out.println("处理请求 " + i + " 成功");
            } else {
                System.out.println("处理请求 " + i + " 失败，超过限流速率");
            }
        }

        // 关闭Redisson客户端连接
        redisson.shutdown();
    }
}
```

在上面的示例中，我们首先创建了一个Redisson客户端连接，并创建了一个`RRateLimiter`对象。`trySetRate()`方法用于设置限流的速率，其中第一个参数是`RateType`，指定限流的类型（OVERALL表示整体限流，针对所有请求），第二个参数是速率（每秒处理的请求数），第三个参数是时间窗口大小（1秒）。

然后，我们模拟处理5个请求，使用`tryAcquire()`方法来尝试获取许可。如果获取许可成功，说明请求在限流速率内，可以继续处理。如果获取许可失败，说明请求超过了限流速率，需要等待下一个时间窗口。

最后，我们关闭了Redisson客户端连接。

需要注意的是，`trySetRate()`方法只需要设置一次，之后无需再次设置。同时，实际应用中可以根据具体需求和业务场景进行适当的配置，比如设置不同的速率和时间窗口大小。

希望这个示例可以帮助你了解如何使用Redisson实现限流功能。如有其他疑问，请随时提问。

# 滑动窗口的限流
Redisson目前没有直接提供原生的滑动窗口限流功能。不过，你可以通过一些自定义的逻辑来实现滑动窗口限流。以下是一个简单的滑动窗口限流的示例代码：

```java
import org.redisson.Redisson;
import org.redisson.api.RScript;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.redisson.connection.ConnectionManager;

import java.util.concurrent.TimeUnit;

public class RedissonSlidingWindowRateLimiter {
    private final String key;
    private final int windowSize;
    private final int maxRequests;
    private final RedissonClient redisson;

    public RedissonSlidingWindowRateLimiter(String key, int windowSize, int maxRequests) {
        this.key = key;
        this.windowSize = windowSize;
        this.maxRequests = maxRequests;
        Config config = new Config();
        config.useSingleServer().setAddress("redis://localhost:6379");
        this.redisson = Redisson.create(config);
    }

    public boolean allowRequest() {
        long currentTime = System.currentTimeMillis();
        long cutoffTime = currentTime - windowSize;

        // 通过Lua脚本执行滑动窗口限流逻辑
        String luaScript = "local time = redis.call('time')\n" +
                          "local timestamp = tonumber(time[1]..'.'..string.sub(time[2], 1, 3))\n" +
                          "local requests = redis.call('zrangebyscore', KEYS[1], timestamp-ARGV[1], timestamp)\n" +
                          "if #requests >= tonumber(ARGV[2]) then\n" +
                          "    return 0\n" +
                          "end\n" +
                          "redis.call('zadd', KEYS[1], timestamp, timestamp)\n" +
                          "redis.call('zremrangebyscore', KEYS[1], 0, timestamp-ARGV[1])\n" +
                          "return 1";
        
        RScript script = redisson.getScript();
        Long result = script.eval(RScript.Mode.READ_WRITE, luaScript, RScript.ReturnType.INTEGER, new String[]{key}, windowSize, maxRequests);

        return result == 1;
    }

    public void close() {
        redisson.shutdown();
    }
}
```

在上面的示例中，我们创建了一个`RedissonSlidingWindowRateLimiter`类，该类使用Lua脚本实现了滑动窗口限流。在`allowRequest()`方法中，我们通过Lua脚本来执行滑动窗口限流逻辑。脚本首先获取当前时间戳，然后查询在窗口内的请求数。如果请求数超过了最大请求数，则返回0表示限流；否则，将当前时间戳添加到有序集合中，并删除窗口之前的时间戳，然后返回1表示允许请求。

需要注意的是，这只是一个简单的滑动窗口限流示例，实际应用中可能还需要考虑分布式场景下的数据一致性和并发控制问题。滑动窗口限流是一个较复杂的算法，根据实际需求可能需要进一步优化和调整。

希望这个示例可以帮助你了解如何使用Redisson实现滑动窗口限流功能。如有其他疑问，请随时提问。