# Day29 · Redis基础（Jedis对象、JadisPool）

## 1. Redis 的 Java 客户端驱动

Redis 在 Java 中有三种主流的**客户端驱动**。在实际开发中，Spring 对这三种驱动进行了整合封装，通常学习 Jedis 了解核心机制即可。

- **Jedis：**每个线程维护一个连接对象，**线程不安全，需连接池配合使用**。
- **Lettuce：**基于 Netty，线程安全，支持异步与响应式，适合多线程共享连接场景
- **Redisson：**线程安全，封装丰富，支持分布式锁、队列等高级功能，适用于分布式需求

---

## 2. Jedis

Java 通过 `Jadis客户端连接对象` 访问 Redis。但该对象**线程不安全**，不应在多线程中共享。建议**统一使用 `JedisPool` 管理连接**。

> **Jedis API 名称与 Redis 原生命令保持一致**。

### 2.1 客户端连接对象（`Jedis`）

| 方法                                   | 含义                   |
| -------------------------------------- | ---------------------- |
| **`new Jedis(String host, int port)`** | 创建连接对象           |
| `auth(String password)`                | 设置连接密码           |
| `select(int index)`                    | 选择数据库索引         |
| `close()`                              | 关闭连接（归还连接池） |

> **`close()` 会归还连接池连接，非池中对象将被销毁。**

---

**例：建立连接对象并与服务器交互**

```java
Jedis jedis = new Jedis("127.0.0.1", 6379); // 建立连接
jedis.auth("123456"); // 设置密码（可选）
jedis.select(0); // 选择数据库（可选）

jedis.set("name", "zhangsan");
String name = jedis.get("name");
jedis.close(); // 关闭连接（归还到连接池）
```

---

## 3. JedisPool（推荐）

JedisPool 是 Jedis 提供的连接池实现，用于解决单连接线程不安全问题。

- 启动时创建若干 Jedis 对象，放入池中备用
- 访问 Redis 时从池中获取 Jedis 对象，用完归还
- 避免频繁创建/销毁连接，提升性能、确保线程安全

### 3.1 配置连接池参数（`JedisPoolConfig`）

| 方法                        | 含义                                                 |
| --------------------------- | ---------------------------------------------------- |
| **`new JedisPoolConfig()`** | 创建连接池配置对象                                   |
| `setMaxTotal(int max)`      | 最大连接数（`借出 + 空闲 ≤ maxTotal`）               |
| `setMaxIdle(int max)`       | 最大空闲连接数（超出将定期销毁）                     |
| `setMinIdle(int min)`       | 最小空闲连接数（不足将补充，但总数不超过最大连接数） |

### 3.2 创建连接池（`JedisPool`）

| 方法                                    | 含义                         |
| --------------------------------------- | ---------------------------- |
| **`new JedisPool(config, host, port)`** | 创建连接池对象（有多种重载） |
| `getResource()`                         | 从连接池获取 Jedis 实例      |
| `close()`                               | 关闭连接池                   |

> 除了直接创建连接池以外，还可以通过封装工具类的方式集中管理配置与连接获取逻辑，便于复用与维护。

---

### 3.3 工具类封装示例：JedisConnectionFactory

```java
public class JedisConnectionFactory {
    private static final JedisPool jedisPool;
    private static final int maxTotal;
    private static final int maxIdle;
    private static final int minIdle;
    private static final int port;
    private static final String host;

    // 静态初始化块：类加载时读取配置并初始化连接池
    static {
        // 从类路径下的 redis.properties 读取连接池配置参数
        ResourceBundle rb = ResourceBundle.getBundle("redis");
        maxTotal = Integer.parseInt(rb.getString("redis.maxTotal"));
        maxIdle = Integer.parseInt(rb.getString("redis.maxIdle"));
        minIdle = Integer.parseInt(rb.getString("redis.minIdle"));
        port = Integer.parseInt(rb.getString("redis.port"));
        host = rb.getString("redis.host");

        // 构造 JedisPoolConfig 对象并设置参数
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(maxTotal);
        config.setMaxIdle(maxIdle);
        config.setMinIdle(minIdle);

        // 创建 JedisPool 连接池对象
        jedisPool = new JedisPool(config, host, port);
    }

    // 对外提供 Jedis 连接对象
    public static Jedis getJedis() {
        return jedisPool.getResource();
    }

    // 关闭连接池（通常用于程序关闭时释放资源）
    public static void closeJedisPool() {
        if (jedisPool != null) {
            jedisPool.close();
        }
    }
}

```

---

## ✅ 总结

- Jedis 是 Redis Java 客户端中最常见的入门驱动
- **多线程下需结合 JedisPool 使用**，避免线程不安全
- 配合 JedisPoolConfig 可灵活配置连接池参数
- Spring Boot 项目常结合 Redis Starter 使用 Lettuce/Redisson，了解 Jedis 有助于掌握连接机制底层原理
